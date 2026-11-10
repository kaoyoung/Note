


```bash
make raspberrypi4_64_defconfig
make 2>&1 | tee build.log
```




```bash
lsblk    # 插卡前後各跑一次，多出來的那個就是 SD 卡，例如 /dev/sdb
sudo dd if=output/images/sdcard.img of=/dev/sdX bs=4M conv=fsync status=progress
```

lsblk、du、df、root
