# Part 1
## Build and run the implementation
### Step 1: Apply the patch and rebuild the kernel

Add the patch to the linux kernel and rebuild the kernel by the following commands:
```
git clone --depth 1 --branch v5.15 https://github.com/torvalds/linux.git
cd linux/ 
git apply --whitespace=fix r13922129_hw1_kernel.patch 
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig 
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j4
```
### Step 2: Build the KVM host and guest VM according to the steps of `vmhw1.pdf`

Follow the steps of `vmhw1.pdf`  to build the KVM host and guest VM. Use the newly built Image (arch/arm64/boot/Image) when launching the KVM host.
### Step 3: Compile the module (`hw1_module.c`)

In Ubuntu x86, place the Makefile and `hw1_module.c` in the same directory on the machine where you compiled the kernel, and execute the following command. 
```shell
make KDIR=/PATH/TO/kernel-source ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-
```
### Step 4: Transfer files to the KVM host
In the Ubuntu x86, use the following commands to transfer the `hw1_module.ko` and `test_hw1.c` to the KVM host
```shell
scp -P 2222 test_hw1.c root@localhost:/root
scp -P 2222 hw1_module.ko root@localhost:/root
```
Note that you have to use `dhclient` to open the internet in KVM.
###  Step 5: Compile `test_hw1.c`
In the KVM host, use the following command the compile the `test.c`
```shell
gcc test_hw1.c -o test
```
### Step 6: Transfer the `hw1_module.ko` and `test` to the guest VM
In the KVM host, use the following commands to transfer the `hw1_module.ko` and `test` to the guest VM
```shell
scp -P 2222 hw1_module.ko test root@localhost:/root
```
Note that you have to use `dhclient` to open the internet in the guest VM.
### Step 7: Install the `hw1_module.ko` 
In the Guest OS,  install the compiled kernel module:
```bash
insmod hw1_module.ko 
ls /dev/hw1_module # confirm device node exists
```
### Step 8: Using `test` to pin the CPU
In the Guest OS, run the test program to trigger the pinning:
```shell
./test
```
## Concept of Design
First the user-space program (`test_hw1.c`) communicates with the guest kernel module (`hw1_module`) through standard file operations (`read` and `write` system calls) on a device node. When the user program writes to the device, the `hw1_module_write()` function is invoked, and using `copy_from_user()` to retrieve the requested CPU mask from user space. Conversely, when the user program reads from the device, the `hw1_module_read()` function uses `copy_to_user()` to return the current CPU mask from the kernel space back to the user space.
Second I use the hypervisor call (`hvc #0`) to let the guest VM communicate with the KVM host. Following the SMCCC specification, two custom function IDs are defined:
```c
#define HW1_HVC_SET_AFFINITY ARM_SMCCC_CALL_VAL( \
                               ARM_SMCCC_FAST_CALL, \
                               ARM_SMCCC_SMC_64, \
                               ARM_SMCCC_OWNER_OEM, \
                               0x0000  \
                           )

#define HW1_HVC_GET_AFFINITY ARM_SMCCC_CALL_VAL( \
                               ARM_SMCCC_FAST_CALL, \
                               ARM_SMCCC_SMC_64, \
                               ARM_SMCCC_OWNER_OEM, \
                               0x0001  \
                           )
```

Third, these hypercalls are intercepted at the beginning of `kvm_hvc_call_handler()` in `arch/arm64/kvm/hypercalls.c`. Two functions handle the respective VM exits. The first function, `hw1_set_vcpu_affinity()`, takes the cpumask value passed by the guest via register `x1`. It allocates a `cpumask_var_t` using `alloc_cpumask_var()`, clears it, and then it calls `cpumask_set_cpu()` to mark the corresponding core. The host thread's `task_struct` is obtained via `get_pid_task(vcpu->pid, PIDTYPE_PID)`, after which `set_cpus_allowed_ptr()` is called to apply the new affinity. Both the `task_struct` reference and the allocated cpumask are released afterwards via `put_task_struct()` and `free_cpumask_var()` respectively. The second function, `hw1_get_vcpu_affinity()`, likewise retrieves the `task_struct` via `get_pid_task(vcpu->pid, PIDTYPE_PID)`. It then iterates over 8 CPU cores, using `cpumask_test_cpu()` to reconstruct the current affinity as an 8-bit mask. After releasing the reference with `put_task_struct()`, the mask is returned to the guest via register `x0`.
# Part 2
## How to do the experiment
Since the guest OS is treated as an independent thread under the QEMU process, we cam get the CPU affinity of this thread by observing `Cpus_allowed_list` field in the `/proc/<qemu-pid>/task/<tid>/status` file on the KVM Host.
The core steps of the experiment are:
1. Record the CPU affinity status of all QEMU threads before triggering the pinning.
2. Execute the test program (`./test`) inside the Guest OS to passing the CPU mask.
3. Use the QEMU Monitor (`info cpus`) to find the host thread ID (TID) corresponding to the guest vCPU that executed the test program.
4. Record the affinity status of all QEMU threads on the KVM Host and compare it with the initial status (before pinning) using the `diff` command.
5. If only the status of that specific vCPU TID changes, while the other threads remain unchanged. It demonstrates that the dynamic pinning mechanism operates successfully.
## How to reproduce the experiment
### Step 1: Start the guest VM and load the kernel module
In the Guest OS, boot up and insert the compiled kernel module:
```bash
insmod hw1_module.ko
```

### Step 2: In the KVM host, record the vCPU thread affinity before pinning
Go back to the KVM host and find the PID of the QEMU process:
```bash
ps aux | grep "qemu-system-aarch64"
```

Execute the `test_cpu_affinity_hw1.sh` script (replace `<qemu-pid>` with the actual PID of the QEMU) to write the pre-pinning state to `test_before.txt`:

```shell
#!/bin/bash
for tid in /proc/<qemu-pid>/task/*; do
      tid_num=$(basename $tid)
      t_name=$(cat /proc/<qemu-pid>/task/$tid_num/comm 2>/dev/null)
      cpus_list=$(cat /proc/<qemu-pid>/task/$tid_num/status 2>/dev/null \
               | grep "^Cpus_allowed_list:" | awk '{print $2}')
      echo "TID: $tid_num | Name: $t_name | CPU List: $cpus_list"
done > test_before.txt
```

Last run `./test_cpu_affinity_hw1.sh`

### Step 3: Trigger CPU Pinning (Guest OS)
In the Guest OS, run the test program to trigger the pinning:
```shell
./test
```

### Step 4: Identify the mapping of vCPU to Host TID 
In the Guest OS, press `Ctrl + A` followed by `c` to enter the QEMU Monitor. Enter the following command to get the mapping between the Guest vCPUs and the Host Thread IDs:
```shell
(qemu) info cpus
```
### Step 5: Record the vCPU thread affinity after pinning
Return to the KVM Host, modify the `test_cpu_affinity_hw1.sh` script to redirect the output to `test_after.txt`, and execute the script to capture the state after pinning:

```shell
# Execute the same shell script as in Step 2, but output to test_after.txt bash 
./test_cpu_affinity_hw1.sh
```

### Step 6: Compare the vCPU result
Use the `diff` command to get the changes between the two states:
```shell
diff test_before.txt test_after.txt
```

## Conclusion
In this experiment, I used the `test_hw1.c` program within the Guest OS (configured with 8 vCPUs) to set the host CPU affinity. The target `cpumask` was set to `50` in decimal, which corresponds to `00110010` in binary (targeting CPU cores 1, 4, and 5). Note that the KVM host is also an 8-core machine.

To verify the pinning mechanism, I captured the QEMU thread states on the KVM host before and after the execution. The picture below shows the output of `diff test_before.txt test_after.txt`.
![[VM_HW1_TEST_1.png]]
The `diff` result clearly shows that the `Cpus_allowed_list` for Thread ID (TID) `5478` successfully changed from `0-7` (allowing execution on all cores) to `1,4-5`.

To prove that TID `5478` is indeed the vCPU thread that triggered the hypercall, I accessed the QEMU monitor to check the thread mapping.
![[VM_HW1_TEST_2.png]]
As shown in the picture above, executing `info cpus` in the QEMU monitor confirms that Guest vCPU #3 maps exactly to Host `thread_id=5478`. This perfectly aligns with our host-side `diff` observation, providing solid evidence that the guest program successfully modified its host CPU affinity via our implemented KVM hypercall mechanism.