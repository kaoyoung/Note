# Concept
We need to turn off the swap and move the data in swap in main memory or disk first. Next create the `/swapfile` and change its mode. Last initialize a partition or file as swap area and trun on the swap function.
# Procedure
## The command
```Shell
# turn off the swap function
sudo swapoff -a

# create new partition
sudo dd if=/dev/zero of=/swapfile bs=1G count=16

# Set the correct permissions
sudo chmod 0600 /swapfile

# initialize a partition or file as swap area
sudo mkswap /swapfile

# turn on the swap function
sudo swapon /swapfile
```
## Check it
```Shell
grep Swap /proc/meminfo
```
## Make it permanent
Add this to `/etc/fstab`
```Shell
/swapfile none swap sw 0 0
```
