# 整體架構
![[nested_kvm_architecture.png]]

### step 1: (compile qemu)
```text
git clone https://gitlab.com/qemu-project/qemu.git 
cd qemu/ 
git checkout tags/v7.0.0
./configure --target-list=aarch64-softmmu --disable-werror
make -j4
sudo make install
```

可能遇到錯誤:
- ERROR: GNU make (make) not found
```
sudo apt update
sudo apt install build-essential
```
- ERROR: Cannot find Ninja
```
sudo apt-get install ninja-build
```
- ERROR: pkg-config binary 'pkg-config' not found
```
sudo apt-get install pkg-config
```
- ERROR: glib-2.56 gthread-2.0 is required to compile QEMU
```
sudo apt-get update
sudo apt-get install libglib2.0-dev
```
- Run-time dependency pixman-1 found: NO (tried pkgconfig) ../meson.build:463:2: ERROR: Dependency "pixman-1" not found, tried pkgconfig
```
sudo apt update
sudo apt install libpixman-1-dev
```

說明
- `git checkout tags/v7.0.0` : `tags/v7.0.0` 表示去 `tags` 資料夾下找 `v7.0.0` 的檔案。
- `./configure --target-list=aarch64-softmmu --disable-werror` : 開啟 `-Werror` 代表編譯器將 `Warning` 視為 `Error` 。
- `sudo make install` : 做四件事
	1. 複製執行檔：把編譯出來的二進位檔（Binary）搬到 `/usr/local/bin`。
	2. 複製函式庫：把相關的 `.so` 或 `.a` 檔搬到 `/usr/local/lib`。
	3. 安裝說明文件：把 `man` 手冊搬到 `/usr/share/man`。
	4. 設定權限：確保這些檔案可以被系統正確執行。
>`usr/local/bin` : 存放你自己下載、編譯並安裝的「可執行程式」（Binaries）。
>`usr/share/man` : 存放軟體的「說明文件」（Manual pages）。

可以用以下方式測
```
which qemu-system-aarch64 
# 應該印出 /usr/local/bin/qemu-system-aarch64 

qemu-system-aarch64 --version 
# 應該印出 QEMU emulator version 7.0
```


### step 2: (compile Linux/KVM host)
```
sudo apt install gcc-aarch64-linux-gnu
git clone --depth 1 --branch v5.15 https://github.com/torvalds/linux.git
cd linux
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j4
```

可能遇到錯誤:
- `/bin/sh: 1: flex: not found`
```
sudo apt install flex
```
- `/bin/sh: 1: bison: not found`
```
sudo apt-get install bison
```
- `fatal error: openssl/bio.h: No such file or directory`
```
sudo apt-get install libssl-dev
```

說明:
- `git clone --depth 1 --branch v5.15 https://github.com/torvalds/linux.git` : `--depth 1` 代表只要下載最新的一次更新紀錄就好。
- `make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig` : `make defconfig` 根據官方預設的參數，自動生成一份基本的編譯設定檔。`ARCH=arm64` 告訴編譯系統，我們要編譯出來的系統是給 64 位元 ARM 架構 (ARM64) 的設備使用的。`CROSS_COMPILE=aarch64-linux-gnu-` 為指定「交叉編譯器」的前綴字。
- `make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j4`

```
qemu-img create -f raw cloud.img 25g 
mkfs.ext4 cloud.img 
mount cloud.img /mnt 
tar xvf ./ubuntu-20.04-server-cloudimg-arm64-root.tar.xz -C /mnt 
sync 
sudo touch /mnt/etc/cloud/cloud-init.disabled
```

說明:
- `qemu-img create -f raw cloud.img 25g` : `qemu-img` 是 QEMU 虛擬機的磁碟工具。 `-f raw` 指令格式為 raw。
- `mkfs.ext4 cloud.img` : 將 `cloude.img` 這映像檔格式化成 ext4 檔案系統，讓他可以存放檔案。
- `mount cloud.img /mnt` : 將 `cloude.img` 這映像檔掛載到 `/mnt` 目錄，這樣就可以向操作一般目錄一樣讀也它的內容。
- `sync` : 強制將記憶體中尚未寫入磁碟的資料全部刷新。

開啟 `/mnt/etc/passwd` 並把第一行改成
```
root::0:0:root:/root:/bin/bash
```
- 把第二個 column 的 x 拿掉，代表 root 不用密碼登入

最後
```
umount /mnt
```

### step 3: (run KVM host)
```
./run-kvm.sh -k $PATH_TO_KVM_HOST_IMAGE -i $PATH_TO_YOUR_cloud.img
```


sudo apt purge snapd 
sudo apt autoremove


`test.c` 中的 `int pin(int fd, u_int8_t to_pin)` 的 `value` 改 `to_pin`