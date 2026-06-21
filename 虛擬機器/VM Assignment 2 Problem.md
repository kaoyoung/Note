# Problem 1
```bash
TOOLS_PATH=~/.shrinkwrap/build/build/cca-3world/buildroot/host/sbin
$TOOLS_PATH/e2fsck -fp rootfs.ext2 
$TOOLS_PATH/resize2fs rootfs.ext2 256M
```
用以下指令替代
```bash
docker run --rm \
  -v $HOME/.shrinkwrap/package/cca-3world:/work \
  ubuntu:24.04 \
  bash -c "e2fsck -fp /work/rootfs.ext2; resize2fs /work/rootfs.ext2 256M"
```
