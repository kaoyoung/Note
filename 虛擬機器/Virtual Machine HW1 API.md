# API 探討
## hw1_module
### struct 定義
```c
static int major; 
static struct class *hw1_module_class; 
static struct device *hw1_module_device; 
```

- `major` : Linux 用來識別驅動程式的號碼。
	- `minor` : 識別具體的裝置實例。
- `struct class` : 
在 `include/linux/device/class.h` 中
```c
/**
 * struct class - device classes
 * @name:	Name of the class.
 * @owner:	The module owner.
 * @class_groups: Default attributes of this class.
 * @dev_groups:	Default attributes of the devices that belong to the class.
 * @dev_kobj:	The kobject that represents this class and links it into the hierarchy.
 * @dev_uevent:	Called when a device is added, removed from this class, or a
 *		few other things that generate uevents to add the environment
 *		variables.
 * @devnode:	Callback to provide the devtmpfs.
 * @class_release: Called to release this class.
 * @dev_release: Called to release the device.
 * @shutdown_pre: Called at shut-down time before driver shutdown.
 * @ns_type:	Callbacks so sysfs can detemine namespaces.
 * @namespace:	Namespace of the device belongs to this class.
 * @get_ownership: Allows class to specify uid/gid of the sysfs directories
 *		for the devices belonging to the class. Usually tied to
 *		device's namespace.
 * @pm:		The default device power management operations of this class.
 * @p:		The private data of the driver core, no one other than the
 *		driver core can touch this.
 *
 * A class is a higher-level view of a device that abstracts out low-level
 * implementation details. Drivers may see a SCSI disk or an ATA disk, but,
 * at the class level, they are all simply disks. Classes allow user space
 * to work with devices based on what they do, rather than how they are
 * connected or how they work.
 */
struct class {
	const char		*name;
	struct module		*owner;

	const struct attribute_group	**class_groups;
	const struct attribute_group	**dev_groups;
	struct kobject			*dev_kobj;

	int (*dev_uevent)(struct device *dev, struct kobj_uevent_env *env);
	char *(*devnode)(struct device *dev, umode_t *mode);

	void (*class_release)(struct class *class);
	void (*dev_release)(struct device *dev);

	int (*shutdown_pre)(struct device *dev);

	const struct kobj_ns_type_operations *ns_type;
	const void *(*namespace)(struct device *dev);

	void (*get_ownership)(struct device *dev, kuid_t *uid, kgid_t *gid);

	const struct dev_pm_ops *pm;

	struct subsys_private *p;
};
```

- `struct device`

在 `include/linux/device.h` 中
```c
/**
 * struct device - The basic device structure
 * @parent:	The device's "parent" device, the device to which it is attached.
 * 		In most cases, a parent device is some sort of bus or host
 * 		controller. If parent is NULL, the device, is a top-level device,
 * 		which is not usually what you want.
 * @p:		Holds the private data of the driver core portions of the device.
 * 		See the comment of the struct device_private for detail.
 * @kobj:	A top-level, abstract class from which other classes are derived.
 * @init_name:	Initial name of the device.
 * @type:	The type of device.
 * 		This identifies the device type and carries type-specific
 * 		information.
 * @mutex:	Mutex to synchronize calls to its driver.
 * @lockdep_mutex: An optional debug lock that a subsystem can use as a
 * 		peer lock to gain localized lockdep coverage of the device_lock.
 * @bus:	Type of bus device is on.
 * @driver:	Which driver has allocated this
 * @platform_data: Platform data specific to the device.
 * 		Example: For devices on custom boards, as typical of embedded
 * 		and SOC based hardware, Linux often uses platform_data to point
 * 		to board-specific structures describing devices and how they
 * 		are wired.  That can include what ports are available, chip
 * 		variants, which GPIO pins act in what additional roles, and so
 * 		on.  This shrinks the "Board Support Packages" (BSPs) and
 * 		minimizes board-specific #ifdefs in drivers.
 * @driver_data: Private pointer for driver specific info.
 * @links:	Links to suppliers and consumers of this device.
 * @power:	For device power management.
 *		See Documentation/driver-api/pm/devices.rst for details.
 * @pm_domain:	Provide callbacks that are executed during system suspend,
 * 		hibernation, system resume and during runtime PM transitions
 * 		along with subsystem-level and driver-level callbacks.
 * @em_pd:	device's energy model performance domain
 * @pins:	For device pin management.
 *		See Documentation/driver-api/pin-control.rst for details.
 * @msi_lock:	Lock to protect MSI mask cache and mask register
 * @msi_list:	Hosts MSI descriptors
 * @msi_domain: The generic MSI domain this device is using.
 * @numa_node:	NUMA node this device is close to.
 * @dma_ops:    DMA mapping operations for this device.
 * @dma_mask:	Dma mask (if dma'ble device).
 * @coherent_dma_mask: Like dma_mask, but for alloc_coherent mapping as not all
 * 		hardware supports 64-bit addresses for consistent allocations
 * 		such descriptors.
 * @bus_dma_limit: Limit of an upstream bridge or bus which imposes a smaller
 *		DMA limit than the device itself supports.
 * @dma_range_map: map for DMA memory ranges relative to that of RAM
 * @dma_parms:	A low level driver may set these to teach IOMMU code about
 * 		segment limitations.
 * @dma_pools:	Dma pools (if dma'ble device).
 * @dma_mem:	Internal for coherent mem override.
 * @cma_area:	Contiguous memory area for dma allocations
 * @dma_io_tlb_mem: Pointer to the swiotlb pool used.  Not for driver use.
 * @archdata:	For arch-specific additions.
 * @of_node:	Associated device tree node.
 * @fwnode:	Associated device node supplied by platform firmware.
 * @devt:	For creating the sysfs "dev".
 * @id:		device instance
 * @devres_lock: Spinlock to protect the resource of the device.
 * @devres_head: The resources list of the device.
 * @knode_class: The node used to add the device to the class list.
 * @class:	The class of the device.
 * @groups:	Optional attribute groups.
 * @release:	Callback to free the device after all references have
 * 		gone away. This should be set by the allocator of the
 * 		device (i.e. the bus driver that discovered the device).
 * @iommu_group: IOMMU group the device belongs to.
 * @iommu:	Per device generic IOMMU runtime data
 * @removable:  Whether the device can be removed from the system. This
 *              should be set by the subsystem / bus driver that discovered
 *              the device.
 *
 * @offline_disabled: If set, the device is permanently online.
 * @offline:	Set after successful invocation of bus type's .offline().
 * @of_node_reused: Set if the device-tree node is shared with an ancestor
 *              device.
 * @state_synced: The hardware state of this device has been synced to match
 *		  the software state of this device by calling the driver/bus
 *		  sync_state() callback.
 * @can_match:	The device has matched with a driver at least once or it is in
 *		a bus (like AMBA) which can't check for matching drivers until
 *		other devices probe successfully.
 * @dma_coherent: this particular device is dma coherent, even if the
 *		architecture supports non-coherent devices.
 * @dma_ops_bypass: If set to %true then the dma_ops are bypassed for the
 *		streaming DMA operations (->map_* / ->unmap_* / ->sync_*),
 *		and optionall (if the coherent mask is large enough) also
 *		for dma allocations.  This flag is managed by the dma ops
 *		instance from ->dma_supported.
 *
 * At the lowest level, every device in a Linux system is represented by an
 * instance of struct device. The device structure contains the information
 * that the device model core needs to model the system. Most subsystems,
 * however, track additional information about the devices they host. As a
 * result, it is rare for devices to be represented by bare device structures;
 * instead, that structure, like kobject structures, is usually embedded within
 * a higher-level representation of the device.
 */
```

```c
static int hw1_module_open(struct inode *inode, struct file *file) {
    return 0;
}
```

- `struct inode` : 代表檔案系統節點（含裝置號碼 `i_rdev`）
在 `include/linux/fs.h` 中
```c
/*
 * Keep mostly read-only and often accessed (especially for
 * the RCU path lookup and 'stat' data) fields at the beginning
 * of the 'struct inode'
 */
struct inode {
	umode_t			i_mode;
	unsigned short		i_opflags;
	kuid_t			i_uid;
	kgid_t			i_gid;
	unsigned int		i_flags;
```

- `sturct file` : 代表一個開啟中的檔案實例
在 `include/linux/fs.h` 中
```
struct file {
	union {
		struct llist_node	fu_llist;
		struct rcu_head 	fu_rcuhead;
	} f_u;
	struct path		f_path;
	struct inode		*f_inode;	/* cached value */
	const struct file_operations	*f_op;

	/*
	 * Protects f_ep, f_flags.
	 * Must not be taken from IRQ context.
	 */
	spinlock_t		f_lock;
	enum rw_hint		f_write_hint;
	atomic_long_t		f_count;
	unsigned int 		f_flags;
	fmode_t			f_mode;
	struct mutex		f_pos_lock;
	loff_t			f_pos;
	struct fown_struct	f_owner;
	const struct cred	*f_cred;
	struct file_ra_state	f_ra;

	u64			f_version;
```

```c
static struct file_operations fops = {
    .owner = THIS_MODULE,
    .open = hw1_module_open,
    .release = hw1_module_release,
    .read = hw1_module_read,
    .write = hw1_module_write,
};
```

- `struct file_operation` : 定義所有 open/read/write/ioctl 等函式指標

在 `include/linux/fs.h` 中
```c
struct file_operations {
	struct module *owner;
	loff_t (*llseek) (struct file *, loff_t, int);
	ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
	ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
	ssize_t (*read_iter) (struct kiocb *, struct iov_iter *);
	ssize_t (*write_iter) (struct kiocb *, struct iov_iter *);
	int (*iopoll)(struct kiocb *kiocb, bool spin);
	int (*iterate) (struct file *, struct dir_context *);
	int (*iterate_shared) (struct file *, struct dir_context *);
	__poll_t (*poll) (struct file *, struct poll_table_struct *);
	long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);
	long (*compat_ioctl) (struct file *, unsigned int, unsigned long);
	int (*mmap) (struct file *, struct vm_area_struct *);
	unsigned long mmap_supported_flags;
	int (*open) (struct inode *, struct file *);
	int (*flush) (struct file *, fl_owner_t id);
	int (*release) (struct inode *, struct file *);
	int (*fsync) (struct file *, loff_t, loff_t, int datasync);
```

>[!note] Parameter explain
>- `struct inode *inode` : The physical device node or file itself. It stores Major/Minor numbers, permissions, device type.
>- `struct file *file` : The open file instance. It stores `loff_t`, open flags.
>- `char __user *buffer` : The address in User Space memory where data should be sent to or taken from.
>- `size_t count` : This is the requested size. It tells your driver how many bytes the user _wants_ to read or write. Your driver should respect this limit to avoid buffer overflows.
>- `loff_t *offset` : This tracks the current position (pointer) within the file.

>[!note] The data flows of `.write`
>1. **User Space:** Calls `write(fd, "hello", 5);`
>2. **System Call Interface:** The kernel receives this request and finds the `struct file` associated with `fd`.
>3. **Your Driver:** The kernel calls your `hw1_module_write`. It passes the `file` pointer, the address of `"hello"`, the number `5`, and the current file position.
>4. **Completion:** Your function moves the data into your hardware/buffer and returns the number of bytes actually processed.

>[!note] The data flows of `.open`
>1. **User Space:** The C program calls `open("/dev/hw1_device", O_RDWR)`.
>2. **System Call Interface:** The call crosses the boundary from user space into kernel space.
>3. **Kernel VFS (Virtual File System):** The kernel looks at the file `/dev/hw1_device` and sees that it is a "Character Device" with a specific Major Number.
>4. **Lookup:** The kernel looks up that Major Number in its internal table of registered drivers and finds the `fops` structure you defined.
>5. **Execution:** The kernel sees that your `.open` pointer points to `hw1_module_open`. It creates a new `struct file` (the session) and passes it to your function.
>6. **Return:** If your function returns `0` (success), the kernel gives the user-space program an integer (the File Descriptor, or `fd`, like `3` or `4`).

>[!note] The data flows of `.release`
>1. **User Space:** The C program finishes its work and calls `close(fd)`.
>2. **System Call Interface:** The request jumps back into kernel space.
>3. **Kernel Check:** The kernel looks at the `struct file` associated with that `fd`. It checks a "reference count" to see how many processes are currently using it (in case the process duplicated the file descriptor).
>4. **Execution:** If this `close()` call drops the reference count to zero (meaning it's the very last close for that session), the kernel looks at your `fops` structure again and executes `hw1_module_release`.

### API 解釋
- `major = register_chrdev(0, DEVICE_NAME, &fops);` : 

在 `include/linux/fs.h` 中
```c
static inline int register_chrdev(unsigned int major, const char *name,
				  const struct file_operations *fops)
{
	return __register_chrdev(major, 0, 256, name, fops);
}
```

在 `fs/char_dev.c` 中
```c
/**
 * __register_chrdev() - create and register a cdev occupying a range of minors
 * @major: major device number or 0 for dynamic allocation
 * @baseminor: first of the requested range of minor numbers
 * @count: the number of minor numbers required
 * @name: name of this range of devices
 * @fops: file operations associated with this devices
 *
 * If @major == 0 this functions will dynamically allocate a major and return
 * its number.
 *
 * If @major > 0 this function will attempt to reserve a device with the given
 * major number and will return zero on success.
 *
 * Returns a -ve errno on failure.
 *
 * The name of this device has nothing to do with the name of the device in
 * /dev. It only helps to keep track of the different owners of devices. If
 * your module name has only one type of devices it's ok to use e.g. the name
 * of the module here.
 */
int __register_chrdev(unsigned int major, unsigned int baseminor,
		      unsigned int count, const char *name,
		      const struct file_operations *fops)
{
```

- `hw1_module_class = class_create(THIS_MODULE, DEVICE_NAME);` : 

在 `include/linux/device/class.h` 中
```c
/**
 * class_create - create a struct class structure
 * @owner: pointer to the module that is to "own" this struct class
 * @name: pointer to a string for the name of this class.
 *
 * This is used to create a struct class pointer that can then be used
 * in calls to device_create().
 *
 * Returns &struct class pointer on success, or ERR_PTR() on error.
 *
 * Note, the pointer created here is to be destroyed when finished by
 * making a call to class_destroy().
 */
#define class_create(owner, name)		\
({						\
	static struct lock_class_key __key;	\
	__class_create(owner, name, &__key);	\
})
```
在 `drivers/base/class.c` 中
```c
/**
 * __class_create - create a struct class structure
 * @owner: pointer to the module that is to "own" this struct class
 * @name: pointer to a string for the name of this class.
 * @key: the lock_class_key for this class; used by mutex lock debugging
 *
 * This is used to create a struct class pointer that can then be used
 * in calls to device_create().
 *
 * Returns &struct class pointer on success, or ERR_PTR() on error.
 *
 * Note, the pointer created here is to be destroyed when finished by
 * making a call to class_destroy().
 */
struct class *__class_create(struct module *owner, const char *name,
			     struct lock_class_key *key)
{
	struct class *cls;
	int retval;

	cls = kzalloc(sizeof(*cls), GFP_KERNEL);
	if (!cls) {
		retval = -ENOMEM;
		goto error;
	}
```

- `hw1_module_device = device_create(hw1_module_class, NULL, MKDEV(major, 0), NULL, DEVICE_NAME);` : 讓系統自動在 `/dev` 目錄下幫你生出一個設備節點（Device Node）

在 `include/linux/device.h`
```c
__printf(5, 6) struct device *
device_create(struct class *cls, struct device *parent, dev_t devt,
	      void *drvdata, const char *fmt, ...);
```
在 `drivers/base/class.c` 中
```c
/**
 * __class_create - create a struct class structure
 * @owner: pointer to the module that is to "own" this struct class
 * @name: pointer to a string for the name of this class.
 * @key: the lock_class_key for this class; used by mutex lock debugging
 *
 * This is used to create a struct class pointer that can then be used
 * in calls to device_create().
 *
 * Returns &struct class pointer on success, or ERR_PTR() on error.
 *
 * Note, the pointer created here is to be destroyed when finished by
 * making a call to class_destroy().
 */
struct class *__class_create(struct module *owner, const char *name,
			     struct lock_class_key *key)
{
	struct class *cls;
	int retval;

	cls = kzalloc(sizeof(*cls), GFP_KERNEL);
	if (!cls) {
		retval = -ENOMEM;
		goto error;
	}

	cls->name = name;
	cls->owner = owner;
	cls->class_release = class_create_release;

	retval = __class_register(cls, key);
	if (retval)
		goto error;

	return cls;

error:
	kfree(cls);
	return ERR_PTR(retval);
}
EXPORT_SYMBOL_GPL(__class_create);
```

- `module_init()` 跟 `module_exit()` :

1. **檢查**你的 `my_driver_init` 寫法對不對。
2. 幫你的 `my_driver_init` **貼上 `init_module` 的標籤**，讓 Linux 核心找得到它。
3. 加上**安全認證**，允許系統執行它。

在 `include/linux/module.h` 中
```c
/**
 * module_init() - driver initialization entry point
 * @x: function to be run at kernel boot time or module insertion
 *
 * module_init() will either be called during do_initcalls() (if
 * builtin) or at module insertion time (if a module).  There can only
 * be one per module.
 */

/* Each module must use one module_init(). */
#define module_init(initfn)					\
	static inline initcall_t __maybe_unused __inittest(void)		\
	{ return initfn; }					\
	int init_module(void) __copy(initfn)			\
		__attribute__((alias(#initfn)));		\
	__CFI_ADDRESSABLE(init_module, __initdata);

/* This is only required if you want to be unloadable. */
#define module_exit(exitfn)					\
	static inline exitcall_t __maybe_unused __exittest(void)		\
	{ return exitfn; }					\
	void cleanup_module(void) __copy(exitfn)		\
		__attribute__((alias(#exitfn)));		\
	__CFI_ADDRESSABLE(cleanup_module, __exitdata);

#endif
```

#### THIS_MODULE
- `hw1_module_class = class_create(THIS_MODULE, DEVICE_NAME);` :

在 `include/linux/export.h` 中定義
```c
#ifndef __ASSEMBLY__
#ifdef MODULE
extern struct module __this_module;
#define THIS_MODULE (&__this_module)
#else
#define THIS_MODULE ((struct module *)0)
#endif
```


## host KVM
### `hypercall.c`
#### Calling function
- `smccc_get_function(vcpu)`: 
```c
static inline u32 smccc_get_function(struct kvm_vcpu *vcpu)
{
	return vcpu_get_reg(vcpu, 0);
}
```

- `unsigned long vcpu_get_reg(const struct kvm_vcpu *vcpu, u8 reg_num)`: Hypervisor 用它來偷看 virtual machime 現在的狀態
	- 當虛擬機執行了某個它沒有權限執行的指令（例如想讀寫硬體 I/O），或者發生中斷時，CPU 會觸發一個叫 **VM Exit** 的機制，把控制權交還給 Hypervisor。這時 Hypervisor 就會呼叫 `vcpu_get_reg` 來讀取 vCPU 的暫存器，藉此知道：「虛擬機剛剛到底執行了什麼指令？」、「它想讀取哪個記憶體位址？」
- `void vcpu_set_reg(struct kvm_vcpu *vcpu, u8 reg_num, unsigned long val)`: Hypervisor 用它來修改 virtual machime 現在的狀態
	- 當 Hypervisor 幫虛擬機處理完剛剛的 I/O 請求後，它需要把讀取到的資料「塞」回虛擬機的暫存器裡，並且把虛擬機的程式指標（Program Counter, PC）往下移動一行（也就是加上指令長度），這樣虛擬機恢復執行時，才會以為自己已經成功執行完指令並繼續往下跑。這些修改動作就是靠 `vcpu_set_reg` 完成的。

```c
/*
 * vcpu_get_reg and vcpu_set_reg should always be passed a register number
 * coming from a read of ESR_EL2. Otherwise, it may give the wrong result on
 * AArch32 with banked registers.
 */
static __always_inline unsigned long vcpu_get_reg(const struct kvm_vcpu *vcpu, u8 reg_num)
{
        return (reg_num == 31) ? 0 : vcpu_gp_regs(vcpu)->regs[reg_num];
}

static __always_inline void vcpu_set_reg(struct kvm_vcpu *vcpu, u8 reg_num,
                                unsigned long val)
{
        if (reg_num != 31)
                vcpu_gp_regs(vcpu)->regs[reg_num] = val;
}
```

>[!question] 這些 register 怎麼決定的
>由 SMCCC Convention 決訂
>![[SMCCC_Register_Convention.png]]
>

>[!question] ARM 架構下，不同執行層級之間如何透過 SMC/HVC 通信
>SMCCC (SMC Calling Convention) 定義了這不同層級之間通過 SMC/HVC 通信的方式，在進行這些呼叫時，暫存器 `WO` 或 `X0` 放處理 32 位元的 function ID ，來告訴底層要調用哪一個服務。
>- \[31\]: Call Type (決定呼叫是快速 (Fast) 還是可讓出 (Yielding))
>	- **`1` (Fast Call)**：快速呼叫。這類呼叫是**不可被中斷 (Atomic)** 的
>	- **`0` (Yielding Call)**：可讓出呼叫。這類呼叫可能會執行比較久，且**允許被中斷**或搶佔（Preempted）
>- \[30\]: Calling Convention (參數與回傳值是 32 位元還是 64 位元)
>	- **`1` (SMC64)**：使用 64 位元的暫存器傳遞參數。只有在 AArch64 狀態下才能使用
>	- **`0` (SMC32)**：使用 32 位元的暫存器傳遞參數。無論在 AArch32 還是 AArch64 狀態下都可以使用。
>- \[29:24\]: Owner Entity ID (指定負責處理此呼叫的服務擁有者)
>	- `0x00`：ARM 架構服務（Architecture Service）    
>	- `0x01`：CPU 服務
>	- `0x02`：SIP 服務（Silicon Partner，晶片廠商自定義，如高通、聯發科）
>	- `0x03`：OEM 服務（設備製造商自定義）
>	- `0x04`：標準安全服務（Standard Secure Service，例如 PSCI 電源管理）
>	- `0x30` - `0x3F`：Trusted OS 呼叫（如 OP-TEE 等安全作業系統）
>- \[23:16\]: Reserved (目前保留，必須為零 (MBZ))
>- \[15:0\]: Function Number (該服務下具體要執行的函式編號)
>
>在 linux 中可以去 `include/linux/arm-smccc.h` 去看一些定義和巨集。
#### helper function
- `cpumask_var_t` : 

在 `include/linux/cpumask.h` 中
- `typedef struct cpumask { DECLARE_BITMAP(bits, NR_CPUS); } cpumask_t;`
- `typedef struct cpumask *cpumask_var_t;`
```
/*
 * cpumask_var_t: struct cpumask for stack usage.
 *
 * Oh, the wicked games we play!  In order to make kernel coding a
 * little more difficult, we typedef cpumask_var_t to an array or a
 * pointer: doing &mask on an array is a noop, so it still works.
 *
 * ie.
 *	cpumask_var_t tmpmask;
 *	if (!alloc_cpumask_var(&tmpmask, GFP_KERNEL))
 *		return -ENOMEM;
 *
 *	  ... use 'tmpmask' like a normal struct cpumask * ...
 *
 *	free_cpumask_var(tmpmask);
 *
 *
 * However, one notable exception is there. alloc_cpumask_var() allocates
 * only nr_cpumask_bits bits (in the other hand, real cpumask_t always has
 * NR_CPUS bits). Therefore you don't have to dereference cpumask_var_t.
 *
 *	cpumask_var_t tmpmask;
 *	if (!alloc_cpumask_var(&tmpmask, GFP_KERNEL))
 *		return -ENOMEM;
 *
 *	var = *tmpmask;
 *
 * This code makes NR_CPUS length memcopy and brings to a memory corruption.
 * cpumask_copy() provide safe copy functionality.
 *
 * Note that there is another evil here: If you define a cpumask_var_t
 * as a percpu variable then the way to obtain the address of the cpumask
 * structure differently influences what this_cpu_* operation needs to be
 * used. Please use this_cpu_cpumask_var_t in those cases. The direct use
 * of this_cpu_ptr() or this_cpu_read() will lead to failures when the
 * other type of cpumask_var_t implementation is configured.
 *
 * Please also note that __cpumask_var_read_mostly can be used to declare
 * a cpumask_var_t variable itself (not its content) as read mostly.
 */
```

- `bool alloc_cpumask_var(cpumask_var_t *mask, gfp_t flags)` :

>GFP flag: get free page flag
>These flags tell what memory zones can be used, how hard the allocator should try to find free memory, whether the memory can be accessed by the userspace etc.
>- `GFP-KERNEL`: The calling context must be allowed to sleep.
>- `GFP_ATMOIC`: If you think that accessing memory reserves is justified and the kernel will be stressed unless allocation succeed.

在 `lib/cpumask.c` 中
```c
/**
 * alloc_cpumask_var - allocate a struct cpumask
 * @mask: pointer to cpumask_var_t where the cpumask is returned
 * @flags: GFP_ flags
 *
 * Only defined when CONFIG_CPUMASK_OFFSTACK=y, otherwise is
 * a nop returning a constant 1 (in <linux/cpumask.h>).
 *
 * See alloc_cpumask_var_node.
 */
bool alloc_cpumask_var(cpumask_var_t *mask, gfp_t flags)
{
	return alloc_cpumask_var_node(mask, flags, NUMA_NO_NODE);
}

/**
 * alloc_cpumask_var_node - allocate a struct cpumask on a given node
 * @mask: pointer to cpumask_var_t where the cpumask is returned
 * @flags: GFP_ flags
 *
 * Only defined when CONFIG_CPUMASK_OFFSTACK=y, otherwise is
 * a nop returning a constant 1 (in <linux/cpumask.h>)
 * Returns TRUE if memory allocation succeeded, FALSE otherwise.
 *
 * In addition, mask will be NULL if this fails.  Note that gcc is
 * usually smart enough to know that mask can never be NULL if
 * CONFIG_CPUMASK_OFFSTACK=n, so does code elimination in that case
 * too.
 */
bool alloc_cpumask_var_node(cpumask_var_t *mask, gfp_t flags, int node)
{
	*mask = kmalloc_node(cpumask_size(), flags, node);

#ifdef CONFIG_DEBUG_PER_CPU_MAPS
	if (!*mask) {
		printk(KERN_ERR "=> alloc_cpumask_var: failed!\n");
		dump_stack();
	}
#endif

	return *mask != NULL;
}
```

- `void cpumask_clear(struct cpumask *dstp)` :

在 `include/linux/cpumask.h` 中
```c
/**
 * cpumask_clear - clear all cpus (< nr_cpu_ids) in a cpumask
 * @dstp: the cpumask pointer
 */
static inline void cpumask_clear(struct cpumask *dstp)
{
	bitmap_zero(cpumask_bits(dstp), nr_cpumask_bits);
}
```

- `void cpumask_set_cpu(unsigned int cpu, struct cpumask *dstp)` :

在 `include/linux/cpumask.h` 中
```
/**
 * cpumask_set_cpu - set a cpu in a cpumask
 * @cpu: cpu number (< nr_cpu_ids)
 * @dstp: the cpumask pointer
 */
static inline void cpumask_set_cpu(unsigned int cpu, struct cpumask *dstp)
{
	set_bit(cpumask_check(cpu), cpumask_bits(dstp));
}
```

- `struct task_struct *get_pid_task(struct pid *pid, enum pid_type type)` :

>[!question] Linux 的 `pid_type` 在幹啥
>在 POSIX 中 process 跟 thread 不一樣，但在 linux 中沒有差別，只有 `tast_struct` 。為了解決都是 `task_struct` 但上層需按不同層級 (Thread, Process, Group, Session) 來發送訊號，所以 linux 引入 `struct pid` 和 `pid_type` 來處理。`pid_type` 定義了四種型態
>- PIDTYPE_PID: 這個執行緒本身。
>- PIDTYPE_TGID: 這個 Process（同一程式的所有執行緒）。
>- PIDTYPE_PGID: 這個 Process Group。
>- PIDTYPE_SID: 這個 Session。
>
>`struct pid` 持有一個 list 讓他可以反查那些 `task_struct` 屬於這個 pid 號碼在某個視角下的群組。所以
>- 一個 `task_struct`: 對應多個 pid 號碼。
>- 一個 `pid`: 對應多個 `task_struct`。
>更多細節參考 [[Posix vs. Linux 進程管理]] 。

在 `kernel/pid.c` 中
```c
struct task_struct *get_pid_task(struct pid *pid, enum pid_type type)
{
	struct task_struct *result;
	rcu_read_lock();
	result = pid_task(pid, type);
	if (result)
		get_task_struct(result);
	rcu_read_unlock();
	return result;
}
```

- `int set_cpus_allowed_ptr(struct task_struct *p, const struct cpumask *new_mask)`

在 `kernel/sched/core.c` 中
```c
/*
 * Change a given task's CPU affinity. Migrate the thread to a
 * proper CPU and schedule it away if the CPU it's executing on
 * is removed from the allowed bitmask.
 *
 * NOTE: the caller must have a valid reference to the task, the
 * task must not exit() & deallocate itself prematurely. The
 * call is not atomic; no spinlocks may be held.
 */
static int __set_cpus_allowed_ptr(struct task_struct *p,
				  const struct cpumask *new_mask, u32 flags)
{
	struct rq_flags rf;
	struct rq *rq;

	rq = task_rq_lock(p, &rf);
	return __set_cpus_allowed_ptr_locked(p, new_mask, flags, rq, &rf);
}

int set_cpus_allowed_ptr(struct task_struct *p, const struct cpumask *new_mask)
{
	return __set_cpus_allowed_ptr(p, new_mask, 0);
}
```

- `void put_task_struct(struct task_struct *t)`

在 `include/linux/sched/task.h` 中
```c
static inline void put_task_struct(struct task_struct *t)
{
	if (!refcount_dec_and_test(&t->usage))
		return;

	/*
	 * Under PREEMPT_RT, we can't call __put_task_struct
	 * in atomic context because it will indirectly
	 * acquire sleeping locks. The same is true if the
	 * current process has a mutex enqueued (blocked on
	 * a PI chain).
	 *
	 * In !RT, it is always safe to call __put_task_struct().
	 * Though, in order to simplify the code, resort to the
	 * deferred call too.
	 *
	 * call_rcu() will schedule __put_task_struct_rcu_cb()
	 * to be called in process context.
	 *
	 * __put_task_struct() is called when
	 * refcount_dec_and_test(&t->usage) succeeds.
	 *
	 * This means that it can't "conflict" with
	 * put_task_struct_rcu_user() which abuses ->rcu the same
	 * way; rcu_users has a reference so task->usage can't be
	 * zero after rcu_users 1 -> 0 transition.
	 *
	 * delayed_free_task() also uses ->rcu, but it is only called
	 * when it fails to fork a process. Therefore, there is no
	 * way it can conflict with __put_task_struct().
	 */
	call_rcu(&t->rcu, __put_task_struct_rcu_cb);
}
```

- ` void free_cpumask_var(cpumask_var_t mask)`

在 `include/linux/cpumask.h` 中
```c
static __always_inline void free_cpumask_var(cpumask_var_t mask)
{
}
```

>[!question] 為何 `free_cpumask_var` 在 `include/linux/cpumask.h` 裡面沒做事（是空的）？
>這與 Linux Kernel 的巨集設定 **`CONFIG_CPUMASK_OFFSTACK`** 有關。
>在 Kernel 中，CPU 的數量（`NR_CPUS`）可能很少（例如一般 PC 的 4 核心或 8 核心），也可能非常巨大（例如超級電腦的數千個核心）。這會導致 `cpumask_var_t` 這個資料結構的大小差異極大。
>為了解決這個問題，Kernel 提供了兩種模式，而你看到的空函式是在**未開啟 OFFSTACK** 的情況
>1. 當 `CONFIG_CPUMASK_OFFSTACK` **未開啟** 時（常見於一般系統）
>	- 原理：因為 CPU 數量不多，`cpumask_var_t` 的大小很小，所以它會被定義成一個**陣列 (Array)**。當你宣告它時，記憶體會直接分配在 Stack（堆疊）上或是直接包在其他的 Struct 裡面。    
>	- 為何沒做事：因為記憶體不是透過動態配置（像是 `kmalloc`）來的，而是依附在變數本身的生命週期裡（函式結束或 Struct 被釋放時自然消失）。既然**沒有動態配置記憶體，自然就不需要釋放記憶體**。因此 `free_cpumask_var` 就被寫成一個空函式 `{ }`。   
>2. 當 `CONFIG_CPUMASK_OFFSTACK` **開啟** 時（用於超多核心系統）
>	- **原理**：因為 CPU 數量極多，`cpumask_var_t` 如果放在 Stack 會導致 Stack Overflow。所以它會被定義成一個**指標 (Pointer)**。  
>	- **這時就會做事了**：在這種模式下，`alloc_cpumask_var()` 會呼叫 `kmalloc` 動態要一塊記憶體；而 `free_cpumask_var(cpumask_var_t mask)` 則會呼叫 `kfree(mask)` 來真正釋放記憶體。（這部分的實作通常在 `lib/cpumask.c` 中）。  
>總結來說： Kernel 保留這個空函式是為了 **統一 API 介面**。開發者在寫 driver 或 kernel code 時，只要一律呼叫 `alloc_cpumask_var` 和 `free_cpumask_var` 就好，不用自己寫 `#ifdef` 來判斷現在的系統到底有沒有開啟 `OFFSTACK`。編譯器會在編譯時，自動把這個空函式給最佳化掉，完全不影響效能。

>[!question] `void put_task_struct(struct task_struct *t)` 的用途在哪
>需要連著 `get_pid_task()` 一起看，在 `get_pid_task()` 時，核心會在 RCU (Read-Copy-Update) 臨界區內尋找 PID Hash Table。找到對應的 `task_struct` 後，會執行 `atomic_inc(&task->usage)`，將該結構的參照計數（Reference Count）加 1。
>- **原因：** 在 SMP (對稱多處理) 系統中，當你的程式碼正在執行 `set_cpus_allowed_ptr` 時，該進程可能剛好在另一個 CPU 上呼叫了 `exit()`（例如虛擬機崩潰或被強制關閉）。如果沒有 `atomic_inc` 鎖定其生命週期，`task_struct` 會被核心提早釋放（Free），導致你的程式引發 Use-After-Free 錯誤並觸發 Kernel Panic。
>而 `put_task_struct()` 時會呼叫 `refcount_dec_and_test(&t->usage)` 讓硬體以原子操作將 `usage` 計數減 1，並返回減完後是否為 0。

>RUS (read-copy update):
>Read-copy update (RCU) is a synchronization mechanism that was added to the Linux kernel in October of 2002. RCU achieves scalability improvements by allowing reads to occur concurrently with updates.


---






### ref : https://claude.ai/share/214f5d7a-6a8b-4640-b1c2-64df3ea864d0