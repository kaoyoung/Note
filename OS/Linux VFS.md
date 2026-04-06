### Ref: [Linux 虚拟文件系统四大对象：超级块、inode、dentry、file之间关系](https://zhuanlan.zhihu.com/p/354100369)、[Linux 核心設計: 檔案系統概念及實作手法](https://hackmd.io/@sysprog/linux-file-system)
# File System

>[!question] What's the file system
>操作系統中負責管理和儲存文件的機制，便於對文件進行查找和訪問。

檔案系統負責為用戶
- 存入
- 讀取
- 修改
- 刪除
- 控制權限等
---
# Linux File System

![[Linux_VFS.png]]

從上圖中可以看到， Linux 將檔案系統分為兩層 VFS 跟具體文件系統。 VFS 提供一個抽象的接口讓上層的系統調用層，可以通過他統一調用下層不同的具體文件系統。 VFS 由 super bloxk, inode, dentry, vfsmount 等組成，存在於記憶體中，在系統啟動時建立，關閉時消亡。

---
# VFS 
## 在 linux 架構中的位置

![[VFS_architecture_Linux.png]]

Linux 中的 User 使用 GLIBC (POSIX 標準、GNU C 運行時庫)  作為應用程序的運行時庫，之後由 OS ，將其轉成 SCI (system-call interface) ， SCI 是 OS kernel 定義的系統調用接口，然後進行實際的 I/O 操作。

## User 如何處理文件

>[!question] 每個文件系統都是獨立的，有不同的組織和操作方法，那對用戶如何操作 ?
>VFS 做中間層，用戶跟 VFS 交流， VFS 再跟文件系統交流。 VFS 將 POSIX API 接口和不同儲存設備的具體接口進行分離，使得底層的文件系統、設備類型對上層應用程序透明。例如 read, write 映射到 VFS 中就是  sys_read, sys_write ，接著 VFS 就可以對不同的實際的文件系統進行相對應的操作。上敘的技巧是 "鉤子結構" (Hook) ， VFS 提供一個抽象的 struct 讓每一個具體文件系統把自己的字段和函數填進去。

---
# Linux VFS object

為了對文件系統進行統一的管理和組織， Linux 創建了一個公共根目錄和全局文件系統樹。要訪問一個文件系統中的文件，必須先將這文件系統掛載在全局文件系統樹的某個根目錄下，這掛載的目錄稱文掛載點。


![[Tradition_FS_Disk.png]]

上圖是傳統的文件系統在磁盤上的布局
- 引導塊 (Boot Block) :  檔案系統中最開始的區塊，專門儲存開機所需的引導程式碼（Bootstrap Code）。計算機啟動時，BIOS/UEFI 會載入此處代碼來載入作業系統。若檔案系統非用於開機，該區塊通常保留為空。

## Super Block



## Inode



## Dentry




## File



---
# Dist & File System


