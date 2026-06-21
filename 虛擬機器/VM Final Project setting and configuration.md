# 硬體確認
```bash
ls /dev/kvm # must exist 

grep -c 'vmx\|svm' /proc/cpuinfo # must be > 0 free -h # want ≥ 8 GB free 

fio --name=test --rw=randread --bs=4k --size=1G --numjobs=4 --runtime=10 --group_reporting # Aim: ≥ 200 MB/s sequential read so WS prefetch is meaningful

```
# 檔案結構
```txt
/opt/fcproject/                  # 共用，group 可讀寫
├── bin/
│   └── firecracker              # 二進位，裝一次
├── assets/
│   ├── vmlinux.bin              # kernel，唯讀
│   └── alpine-base.ext4         # 原始 rootfs，唯讀
├── rootfs/
│   └── http-server.ext4         # 裝好 HTTP server 的 rootfs（Week 2 建）
├── snapshots/
│   └── golden/
│       ├── snap.file            # golden snapshot（Week 2 建，之後所有測試都從這 restore）
│       └── snap.mem             # memory file，MAP_PRIVATE，多個 VM 可同時讀
├── scripts/                     # 共用 benchmark 腳本
└── results/                     # 所有人的實驗結果集中存放

~/fc-work/                       # 各自的暫存工作區
└── vms/
    ├── vm0/snap.mem             # N concurrent VM 各自的 mem file（Week 3）
    ├── vm1/snap.mem
    └── ...
```
# 開機
### Terminal 1
```bash
SOCK=/tmp/firecracker-$(whoami).sock
rm -f $SOCK
firecracker --api-sock $SOCK
```
### Terminal 2
```bash
SOCK=/tmp/firecracker-$(whoami).sock

curl -X PUT --unix-socket $SOCK http://localhost/boot-source \
  -H 'Content-Type: application/json' \
  -d '{"kernel_image_path": "/opt/fcproject/assets/vmlinux.bin", "boot_args": "console=ttyS0 reboot=k panic=1 pci=off"}'

curl -X PUT --unix-socket $SOCK http://localhost/drives/rootfs \
  -H 'Content-Type: application/json' \
  -d '{"drive_id": "rootfs", "path_on_host": "/opt/fcproject/assets/ubuntu-22.04.ext4", "is_root_device": true, "is_read_only": true}'

curl -X PUT --unix-socket $SOCK http://localhost/actions \
  -H 'Content-Type: application/json' \
  -d '{"action_type": "InstanceStart"}'
```

# 建立 snapshot
# Terminal 2
```bash
SOCK=/tmp/firecracker-$(whoami).sock

# 暫停 VM
curl -X PATCH --unix-socket $SOCK http://localhost/vm \
  -H 'Content-Type: application/json' \
  -d '{"state": "Paused"}'

# 建立 snapshot
curl -X PUT --unix-socket $SOCK http://localhost/snapshot/create \
  -H 'Content-Type: application/json' \
  -d '{
    "snapshot_type": "Full",
    "snapshot_path": "/opt/fcproject/snapshots/golden/snap.file",
    "mem_file_path": "/opt/fcproject/snapshots/golden/snap.mem"
  }'

# 確認
ls -lh /opt/fcproject/snapshots/golden/
```

# restore
### Terminal 2
關閉當前 firecracker
```bash
pkill -f "firecracker --api-sock /tmp/firecracker-$(whoami).sock"
```
### Terminal 1
```bash
SOCK=/tmp/firecracker-$(whoami).sock
rm -f $SOCK
firecracker --api-sock $SOCK
```
### Terminal 2
```bash
SOCK=/tmp/firecracker-$(whoami).sock

curl -X PUT --unix-socket $SOCK http://localhost/snapshot/load \
  -H 'Content-Type: application/json' \
  -d '{
    "snapshot_path": "/opt/fcproject/snapshots/golden/snap.file",
    "mem_file_path": "/opt/fcproject/snapshots/golden/snap.mem",
    "enable_diff_snapshots": false
  }'

curl -X PATCH --unix-socket $SOCK http://localhost/vm \
  -H 'Content-Type: application/json' \
  -d '{"state": "Resumed"}'
```
# 關機
### Terminal 2
```bash
SOCK=/tmp/firecracker-$(whoami).sock

curl -X PUT --unix-socket $SOCK http://localhost/actions \
  -H 'Content-Type: application/json' \
  -d '{"action_type": "SendCtrlAltDel"}'
```
確認關機
```bash
pgrep -a firecracker
```

# 設定 drop_caches 免密碼 sudo（Week 2 開始需要）
 確定真的有清空快取
```bash
# 用數字看，單位 MB
awk '/Cached/ {print $2}' /proc/meminfo
sudo /usr/local/bin/fc-drop-caches
awk '/Cached/ {print $2}' /proc/meminfo
```



---
# **以下可忽略，單純是安裝 firecracker 的過程跟一些想法，可以當作我在搞笑**

# 建置指令（一個人做，需要 sudo）

```bash
# 建立 group 和目錄
sudo groupadd fcteam
sudo usermod -aG fcteam personA
sudo usermod -aG fcteam personB
sudo usermod -aG fcteam personC

sudo mkdir -p /opt/fcproject/{bin,assets,rootfs,snapshots/golden,scripts,results}
sudo chown -R root:fcteam /opt/fcproject
sudo chmod -R 775 /opt/fcproject

# 安裝 Firecracker
ARCH="$(uname -m)" && FCVER="v1.8.0"
curl -Lo /tmp/firecracker.tgz \
  "https://github.com/firecracker-microvm/firecracker/releases/download/${FCVER}/firecracker-${FCVER}-${ARCH}.tgz"
tar -xz -f /tmp/firecracker.tgz --strip-components=1 \
  "release-${FCVER}-${ARCH}/firecracker-${FCVER}-${ARCH}"
sudo mv "firecracker-${FCVER}-${ARCH}" /opt/fcproject/bin/firecracker
sudo chmod +x /opt/fcproject/bin/firecracker
sudo ln -s /opt/fcproject/bin/firecracker /usr/local/bin/firecracker

# 確認版本
firecracker --version

# 下載 kernel 和 rootfs
ARCH="$(uname -m)"
CI_VERSION="v1.8"

sudo curl -fL -o /opt/fcproject/assets/vmlinux.bin \
  "https://s3.amazonaws.com/spec.ccfc.min/firecracker-ci/${CI_VERSION}/${ARCH}/vmlinux-6.1.82"

sudo curl -fL -o /opt/fcproject/assets/ubuntu-22.04.ext4 \
  "https://s3.amazonaws.com/spec.ccfc.min/firecracker-ci/${CI_VERSION}/${ARCH}/ubuntu-22.04.ext4"

du -h /opt/fcproject/assets/*


sudo chmod 644 /opt/fcproject/assets/*
```

>[!note] 讀寫 `/dev/kvm` 權限
>確認現狀
>```bash
>getfacl /dev/kvm
>```
>加入 user
>```bash
># 方法一：加入 kvm group（最乾淨） 
>sudo setfacl -m u:$(whoami):rw /dev/kvm
>```
>上面這方法重開機或重登入後會失效。永久方法
>```
># 建立 udev rule，每次 /dev/kvm 建立時自動套用 ACL
>sudo tee /etc/udev/rules.d/99-kvm.rules > /dev/null << 'EOF'
>KERNEL=="kvm", GROUP="kvm", MODE="0660", RUN+="/usr/bin/setfacl -m u:personA_username:rw,u:personB_username:rw,u:personC_username:rw /dev/kvm"
>EOF
>```
>立刻套用
>```bash
># 立刻套用（不用重開機）
>sudo udevadm control --reload-rules
>sudo udevadm trigger --name-match=kvm
>```
>確認生效
>```bash
>getfacl /dev/kvm
>```
# 設定 drop_caches 免密碼 sudo（Week 2 開始需要）
```bash
sudo tee /usr/local/bin/fc-drop-caches > /dev/null << 'EOF'
#!/bin/bash
echo 3 > /proc/sys/vm/drop_caches
EOF
sudo chmod +x /usr/local/bin/fc-drop-caches

sudo tee /etc/sudoers.d/fcteam > /dev/null << 'EOF'
%fcteam ALL=(ALL) NOPASSWD: /usr/local/bin/fc-drop-caches
EOF
```

# 硬體檢查
```bash
echo "=== KVM ===" && ls /dev/kvm 
echo "=== CPU virt ===" && grep -c 'vmx\|svm' /proc/cpuinfo
echo "=== RAM ===" && free -h
echo "=== Disk (available) ===" && df -h /
echo "=== Disk type ===" && lsblk -d -o NAME,TYPE,SIZE,ROTA
echo "=== Disk speed ===" && dd if=/dev/zero of=/tmp/test bs=1M count=512 conv=fdatasync 2>&1 && rm /tmp/test
echo "=== OS ===" && uname -r && cat /etc/os-release | grep -E "^(NAME|VERSION)="
```

```bash
fio --name=seqread \
    --filename=/dev/nvme0n1 \   # 直接對磁碟裝置測，不經過 cache
    --rw=read \                  # 只測讀取（不寫）
    --bs=1M \                    # 每次讀 1MB（模擬 prefetch 的大塊讀取）
    --size=4G \                  # 總共讀 4GB 的資料
    --numjobs=1 \                # 一條讀取線，模擬單一 VM
    --direct=1 \                 # 繞過 page cache，確保真的從磁碟讀
    --runtime=10 \               # 跑 10 秒
    --group_reporting
```


# 專案目標
Firecracker 用 snapshot 還原 VM 時，是靠 **prefetch working set**（REAP 的做法）來加速。但當同時還原很多 VM 時（burst），每個 VM 都在搶同一條 NVMe 頻寬，沒有任何協調機制，可能導致 tail latency（P95/P99）急遽增加。

>[!question] 核心問題
> 如果在 userspace 做一個 I/O 排程 daemon，替 N 個同時還原的 VM 排隊讀取 working set，能不能讓 tail latency 變好？

過程:
```text
量測 contention（Week 3）
    ↓ 畫出 P99 superlinear degradation curve
設計 daemon（Week 4）
    ↓ socket protocol + mock 驗證
實作三種 policy（Week 5-6）
    ↓ hard freeze，不再加功能
系統性實驗（Week 7-8）
    N={1,2,4,8,16} × Policy={Unscheduled,FIFO,RR,Deadline} × 20 trials
    ↓
分析 + 寫作（Week 9-10）
```
- **接受 negative result**：如果排程沒有改善（例如 NVMe 夠快、contention 不嚴重），結論就是「FaaSnap 的 uncoordinated design 在極端並發下仍然成立」，這也是有效的貢獻。

>[!important] 
>為了避免直接從記憶體還原 VM ，在每次測量還原 N 個 VM 的 latency 需要清除在記憶體的 VM snapshot 檔
>```bash
># 1. 清掉 page cache，確保每次都真的從磁碟讀
>echo 3 > /proc/sys/vm/drop_caches
># 2. 確認 snapshot 檔案不在 cache 裡（要是 0）
>fincore /path/to/snapshot.file
>```

# Firecracker 時間軸
```text
[restore 開始]
      │
      ├─ 讀 snapshot 檔案進 RAM        ← 這裡有大量 I/O
      ├─ 還原 CPU / device 狀態
      ├─ prefetch working set 進記憶體  ← I/O 最集中的地方
      │
[VM 開始執行]
      │
      ├─ 跑你的 function（Python HTTP server）
      ├─ 如果 function 有讀寫檔案，那是 VM 自己的 I/O
      │   → 跟這個專題無關
      │
[first HTTP response] ← 你量的終點
```


# 可能問題
1. **Linux blk-mq**：kernel 的 multi-queue block layer 已經在做 I/O 排程和合併
2. **實際頻寬**：消費級 NVMe 5–7 GB/s，企業級更高；16 個 VM 就算每個 working set 100MB，總共才 1.6GB，在快的 SSD 上不到 1 秒就讀完了



# 可拓展點
1. 