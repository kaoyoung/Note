# Part 1
## 整個 stack 
```bash
你的 x86 機器
  └── shrinkwrap run 起動 FVP（模擬 Armv9 SoC）
       └── FVP 內跑一個 Linux「FVP host」
            └── 這個 host 才會跑 cloud-hypervisor / lkvm
                 └── 啟動 Realm guest
```
## 整個 workflow 
```txt
[build 一次]
  shrinkwrap build  →  產生 rootfs.ext2（裡面有 buildroot Linux）

[setup 一次：把 lkvm 等檔案塞進去]
  mount rootfs.ext2 → cp lkvm/EFI/guest-disk.img → umount

[每次改 cloud-hypervisor 都要重做這個循環]
  ┌─ 在 aarch64 VM 內改 code、build
  ├─ scp binary 回 x86 host
  ├─ mount rootfs.ext2 → cp 新 binary 蓋掉舊的 → umount    ← 只更新 binary
  ├─ shrinkwrap run                                       ← 開 FVP 測
  └─ 在 FVP 內跑 ./cloud-hypervisor ... 驗證
       ↑ Ctrl+] 退出 FVP，回到上面 loop
```

## 開 FVP
```bash
shrinkwrap run cca-3world.yaml --rtvar ROOTFS=~/.shrinkwrap/package/cca-3world/rootfs.ext2
```


### Q1. Realm KVM Interface 
(a) 
I faced two boot errors in part 1. The first was: "Error booting VM: VmBoot(CreateArmRme(ConfigRealm(Invalid argument (os error 22))))". Running `grep -rn "ConfigRealm"` in the `cloud-hypervisor` folder traced the error to `arm_rme_realm_create()` in `hypervisor/src/kvm/mod.rs`. This function calls `vm.enable_cap()` with `cap = KVM_CAP_ARM_RME`, and the `EINVAL` originates from this ioctl being rejected by the v8 host kernel. Comparing `KVM_CAP_ARM_RME` between the two ABIs:
- v7 (`kvm-fork/kvm-bindings/src/arm64/bindings.rs`): `pub const KVM_CAP_ARM_RME: u32 = 300;`
- v8 (`linux/include/uapi/linux/kvm.h`): `#define KVM_CAP_ARM_RME 240`
The cap number was renumbered between v7 and v8, so the v8 kernel does not recognize cap 300 and returns `-EINVAL`. I changed `KVM_CAP_ARM_RME` from `300` to `240` in `kvm-fork/kvm-bindings/src/arm64/bindings.rs`.
The second error was `VmBoot(CpuManager(RecFinalize(VcpuFinalize(Invalid argument (os error 22)))))`. Starting from the variant `VcpuFinalize`, running `grep -rn "VcpuFinalize"` led to `hypervisor/src/kvm/mod.rs`, where `fn vcpu_finalize(feature: i32)` wraps the `KVM_ARM_VCPU_FINALIZE` ioctl error into the `VcpuFinalize` variant. Tracing its callers led to `rec_finalize()`, which passes `KVM_ARM_VCPU_REC as i32` as the feature .
Comparing v7 and v8:
- v7 (`kvm-fork/kvm-bindings/src/arm64/bindings.rs`): `KVM_ARM_VCPU_REC = 8`
- v8 (`linux/arch/arm64/include/uapi/asm/kvm.h`): `HAS_EL2 = 7`, `HAS_EL2_E2H0 = 8` , `REC = 9`
v8 inserted `HAS_EL2_E2H0` at position 8, shifting `REC` to 9. The v7 value 8 is silently interpreted by the v8 kernel as `HAS_EL2_E2H0`, so the ioctl returns `-EINVAL`. I changed `KVM_ARM_VCPU_REC` from `8` to `9` and added `KVM_ARM_VCPU_HAS_EL2_E2H0 = 8` in the same bindings file. After both patches, the Realm guest boots to a login prompt.
(b)
`RMI_REC_ENTER` is used in CCA to execute a vCPU. A REC (Realm Execution Context) is entered via this command ([RMM spec](https://developer.arm.com/documentation/den0137/2-0bet1/) p.784).
(c)
`RMI_RTT_CREATE`, `RMI_DATA_CREATE`, and `RMI_DATA_CREATE_UNKNOWN` are used to create the stage-2 mapping. `RMI_RTT_CREATE` installs a new RTT at a given level of the stage-2 walk for the specified IPA. `RMI_DATA_CREATE` and `RMI_DATA_CREATE_UNKNOWN` install leaf entries binding host-delegated granules to Realm IPAs. Note that `RMI_DATA_CREATE` copies content from a non-secure source, while `RMI_DATA_CREATE_UNKNOWN` does not. The implementations of all three are wrapped in`linux/arch/arm64/include/asm/rmi_cmds.h`.

---
# Part 2








### Q2. Error analysis 
(a) 
`FAR_EL2` (Fault Address Register, EL2): Stores the virtual address that the Realm attempted to access but could not.
`ELR_EL2` (Exception Link Register, EL2): Stores the return address. This address is the PC of the instruction in the Realm that was executing at the moment the exception was taken to EL2.
(b)
Flag register, UARTFR. This register can reflect the current state of the UART, such as whether the transmit FIFO is full, the UART is busy, or data set ready.
(c) 
`void inject_sync_idabort(unsigned long fsc)` is the exact function in RMM that injects this error. Its comment "Inject the Synchronous Instruction or Data Abort into the current REC." is the clear evidence.
### Q3. RMM specification 
(a) 
The IPA space of Realm is separated into two halves: Protected IPA (most significant bit = 0) and Unprotected IPA (most significant bit = 1) (§A5.2.1). The confidentiality and the integrity are guaranteed for the Protected IPA not for the Unprotected IPA (§A2.2.2.2). Only the Protected IPAs have an associated RIPAS (§A5.2.1). Realm software must tolerate Granule Protection Faults (GPFs) on Unprotected IPA access (§A5.2.7).
(b) 
The section A5.2.3 (Realm access to a Protected IPA):
- data access to Protected IPA with RIPAS_EMPTY
- instruction fetch from Protected IPA with RIPAS_EMPTY
- instruction fetch from Protected IPA with RIPAS_DEV
Note that data access or instruction fetch to Protected IPA with RIPAS_RAM or data access  to Protected IPA with RIPAS_DEV may cause SEA if the underlying external abort is restartable or recoverable. 
The section A5.2.7 (Realm access to an Unprotected IPA):
- instruction fetch from an **Unprotected** IPA
- If data access to an Unprotected IPA causes a REC exit due to Data Abort, the host may choose to inject SEA
### Q4. Verifying the fix After applying your fix, run the following experiments on the same kernel image: 
(a) 
The result of `grep "pl011" /proc/iomem` as follows:
```
# grep "pl011" /proc/iomem
09000000-09000fff : pl011@9000000
  09000000-09000fff : 9000000.pl011 pl011@9000000
```
PL011 occupies **`0x09000000`–`0x09000fff`** (4 KB MMIO).
(b) 
```
# dmesg | grep -i "earlycon"
[    0.000000] earlycon: pl11 at MMIO 0x0000800009000000 (options '')
[    0.000000] Kernel command line: console=ttyAMA0 root=/dev/vda2 earlycon=pl011,mmio,0x800009000000
```
The MMIO address is at `0x0000800009000000`, which is different from the address in (a).
(c) 
```
# dmesg | grep -iE "bootconsole|earlycon"
[    0.000000] earlycon: pl11 at MMIO 0x0000800009000000 (options '')
[    0.000000] printk: legacy bootconsole [pl11] enabled
[    0.000000] Kernel command line: console=ttyAMA0 root=/dev/vda2 earlycon=pl011,mmio,0x800009000000
[    3.608532] printk: legacy bootconsole [pl11] disabled
```
The two message is `[    0.000000] printk: legacy bootconsole [pl11] enabled` and `[    3.608532] printk: legacy bootconsole [pl11] disabled`.
(d) 
The dmesg message of Normal VM as follows:
```
# dmesg | grep -iE "bootconsole|earlycon"
[    0.000000] earlycon: pl11 at MMIO 0x0000000009000000 (options '')
[    0.000000] printk: legacy bootconsole [pl11] enabled
[    0.000000] Kernel command line: console=ttyAMA0 root=/dev/vda2 earlycon=pl011,mmio,0x09000000
[    1.520686] printk: legacy bootconsole [pl11] disabled
```
Since this message shows `earlycon: pl11 at MMIO 0x0000000009000000` and `printk: legacy bootconsole [pl11] enabled`, we can ensure that patched cloud-hypervisor still produce a working earlycon for the Normal VM.
### Q5. Realm guest kernel: ioremap path 
(a) 
CPU uses the virtual addresses (MMU is on) and MMIO regions are not included in the kernel's linear map, so there's no VA pointing to the device's PA. Even if MMIO were forced into the linear map, the linear map uses Normal cacheable attributes (CPU cache, reorder, speculate, and merge), which corrupts device behavior. `ioremap()` solves the previous problem by allocating a VA and maps it to the MMIO PA with Device memory attributes.
(b)
Since the IPA space of Realm is divided into two halves: Protected IPA and Unprotected IPA. Only accesses to Unprotected IPA are forwarded to host and MMIO must be emulated by the host, so the address of MMIO must fall in the Unprotected IPA. Therefore `ioremap()` ORs the highest valid IPA bit into the physical address so that the PTE maps the kernel VA to an Unprotected IPA. On the other hand, Normal VM won't do this because its entire IPA space is uniformly visible to host KVM.
(c)
Earlycon uses the fixmap instead of `ioremap()` to map the UART since it runs before the full Memory Management subsystem is up. When fixmap runs, it doesn't have the protected to unprotected translation. Hence, if that address is a protected IPA, earlycon will access protected IPA and trigger an SEA.
### Q6. Host kernel: MMIO fault forwarding 
(a)
`int kvm_handle_guest_abort(struct kvm_vcpu *vcpu)` in the `arch/arm64/kvm/mmu.c`.
(b) 
`HPFAR_EL2`.
KVM reads it from `vcpu->arch.fault.hpfar_el2` via `kvm_vcpu_get_fault_ipa()` in `arch/arm64/include/asm/kvm_emulate.h`.
(c) 
`io_mem_abort()` in the `arch/arm64/kvm/mmio.c`.
(d)
`KVM_EXIT_MMIO`. Since in the `io_mem_abort()`, they wrote 
```c
run->exit_reason        = KVM_EXIT_MMIO;
```
(e) 
No.
HPFAR_EL2 only encodes `IPA[MAX:12]`, since stage-2 faults are reported at page granularity. Before forwarding to userspace, `kvm_handle_guest_abort()` completes the page offset with the low 12 bits from `kvm_vcpu_get_hfar()` (`ipa |= kvm_vcpu_get_hfar(vcpu) & GENMASK(11, 0);`), so the userspace VMM receives a full byte-level IPA. 
In a Realm VM, the guest accesses MMIO via an Unprotected IPA (`0x800009000018` in this assignment), so the IPA in `HPFAR_EL2` carries that high bit. However, cloud-hypervisor's PL011 emulator is registered on `mmio_bus` at the lower-half address `0x09000000`. For the existing MMIO emulation to work without modification, the protected/unprotected selector bit must be cleared by the host before forwarding to userspace.
