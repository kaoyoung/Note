# Compile
### Step 1: compile KVM host kernel (在Ubuntu x86)
```bash
cd linux/ 
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j4
```
確認一下
```bash
ls arch/arm64/boot/Image
```

### Step 2: compile hw1 module
```bash
cd hw1_module/ 
make KDIR=/path/to/linux ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-
```
確認一下
```bash
ls hw1_module.ko
```

### Step 3: Start KVM host
```bash
./run-kvm.sh -k /path/to/linux/arch/arm64/boot/Image -i /path/to/cloud.img
```

### Step 4: put files into KVM host
```bash
scp -P 2222 hw1_module.ko root@localhost:/root
scp -P 2222 test.c root@localhost:/root

scp -P 2222 linux/arch/arm64/boot/Image root@localhost:/root 
scp -P 2222 cloud_inner.img root@localhost:/root

scp -P 2222 run-guest.sh root@localhost:/root
```

### Step 5: go into KVM host & compile test.c
```bash
ssh root@localhost -p 2222
gcc test.c -o test

./run-guest.sh -k Image -i cloud_inner.img
```

### Step 6: go into guest VM and install module
```bash
scp -P 2222 hw1_module.ko root@localhost:/root 
scp -P 2222 test root@localhost:/root

insmod hw1_module.ko
cat /proc/devices         # find hw1_module
mknod 666 /dev/virt_walker
cat /proc/devices         # why we can't find /dev/virt_walker
```
### Step 7: go back to KVM host
```bash
ps aux | grep qemu      # find the pid of qemu and change test_before.sh & test_after.sh
./test_before.sh
```
Where `test_before.sh` is as follow 
```bash
for tid in /proc/658/task/*; do 
	tid_num=$(basename $tid) 
	cpus=$(cat /proc/658/task/$tid_num/status | grep "^Cpus_allowed:" | awk '{print $2}') 
	echo "$tid_num $cpus" 
done > /tmp/before.txt 

cat /tmp/before.txt
```

### Step 8: go to guest OS and run `./test`
```bash
./test
```

### Step 9: go back to KVM host
```bash
./test_after.sh
diff /tmp/before.txt /tmp/after.txt
```
Where `test_after.sh` is as follws
```bash
for tid in /proc/658/task/*; do 
	tid_num=$(basename $tid) 
	cpus=$(cat /proc/658/task/$tid_num/status | grep "^Cpus_allowed:" | awk '{print $2}') 
	echo "$tid_num $cpus" 
done > /tmp/after.txt 

cat /tmp/after.txt
```

>[!question]  Why do need to create a device node ?
>**Key insight:** "Everything is a file". We want to interact with devices just like files.
>User-space application have no way to directly call a function inside kernel memory. The only way user-space programs know how to communicate with the outside world is by reading and writing files using standard file paths. A device node (created via `mknod`) is not a real file that stores data on your hard drive. Instead, think of it as a **magic portal** or a **doorway**.
>- When you create it, you stamp it with a "Major Number" (e.g., 240).    
>- When a user-space program calls `open("/dev/hw1_device")`, the kernel intercepts this.    
>- The kernel looks at the portal, sees the number 240, and says: _"Ah! The user doesn't want to read a text file. They want to talk to the kernel module that owns Major Number 240."_
>Without the device node in `/dev`, your user-space program simply has no path to target, and no "door" to knock on to reach your driver.

>[!question] Can we assign the major number ourselves ?
>In linux development, there are two ways to get a major number
>1. Static Allocation:
>You can hardcore a specific major number into your driver's initialization code.
>```c
>int major_number = 240; 
>int result = register_chrdev(major_number, "hw1_device", &fops); 
>if (result < 0) { 
>	printk(KERN_ALERT "Failed to register major number %d\n", major_number); 
>	return result; 
>}
>```
>2. Dynamic Allocation:
>Because guessing a free number is risky on modern systems with hundreds of drivers, the professional standard is to ask the kernel to assign you an unused number automatically.
>You do this using a different function called `alloc_chrdev_region`, or by passing `0` to `register_chrdev`:
>```c
>// Pass 0 to let the kernel pick! 
>int major_number = register_chrdev(0, "hw1_device", &fops); 
>if (major_number < 0) { 
>	printk(KERN_ALERT "Failed to get a major number\n"); 
>	return major_number; 
>} 
>printk(KERN_INFO "The kernel gave me major number: %d\n", major_number);
>```
>- Because you don't know the number until after you `insmod` the module, you have to check the kernel logs (`dmesg`) to see what number the kernel gave you before you can run your `mknod` command.

>`dmesg` : When turning on the computer, kernel will be loaded into memory and module/driver starts to boost the hardware. In this process, it will print lots of messages. These messages will be written into the ring buffer in the kernel. Note that the size of ring buffer is fixed.

