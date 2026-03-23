參考自[Linux进程占用监控：top、ps、htop工具详解](https://comate.baidu.com/zh/page/1rgeqh6h0ow)
## top
參考自[Unix/Linux TOP 指令使用詳解](https://tigercosmos.xyz/post/2020/04/unix/top-usage/)
### 功能
Linux默认的实时进程监控工具，支持动态排序和交互操作
### 顯示項目
- `PID`: 執行任務的 Process ID
- `USER`: 執行任務的使用者是誰
- `PR`: 任務的優先度 (Priority)
- `NI`: 任務的 Nice Value，負的值代表優先度高，正的值代表優先度低
- `VIRT`: 總共用到多少 kB 虛擬記憶體 (Virtual Memory)
- `RES`: 實體記憶體 (Resident Size) 大小 kB
- `SHR`: 總共用到多少 kB 的共享記憶體 (Shared Memory)
- `S`: 狀態 (Status)
    - R 代表執行中
    - D 代表不可中斷睡眠 (不可被 signal 打斷通常在等 I/O)
    - S 代表睡眠 (可被喚醒)
    - T 中斷中或停止，可能是被 `SIGSTOP` 或 `SIGTSTP` 停止，或是被 degubber 中斷 (ptrace)
    - Z 代表殭屍，通常發生在 Child 已經執行完，等待 Parent 結束或回收
- `%CPU`: 占用到多少 CPU %，注意到一個核心是 100%，所以==多核心是可以超過 100% 的==
- `%MEM`: 占用到多少全部記憶體多少比例
- `TIME+`: 已經執行多少時間
- `COMMAND`: 任務的指令名稱
### 任務排序
- `M`: 照占用記憶體比例排序
- `N`: 照 PID 大小排列
- `P`: 照 CPU 使用多寡排序
- `T`: 照執行時間長短排序
### 一般操作
- Enter/Space: 刷新螢幕 (預設 3 秒)
- `B`: 使用粗體
- `I`: 切換成顯示 CPU 使用比例/全部 CPU 數量
- `Z`: 改變顏色

---
## htop 
### 功能
`htop`是`top`的增强版，支持鼠标操作、颜色高亮、树状视图等，需安装后使用。
### 功能亮點
- **鼠标操作**：点击列标题排序，右键进程可操作（如终止）。
- **树状视图**：按`F5`显示进程父子关系。
- **颜色区分**：高CPU进程标红，高内存进程标黄。

---
## ps 
參考自[Linux ps指令](https://www.hy-star.com.tw/tech/linux/comm/ps.html)
### 功能
`ps`(process state) 用于查看系统当前进程的快照，适合结合排序和过滤分析特定进程。
### 基本語法
> `ps [OPTIONS]`
### options簡寫
- -a：顯示所有運行中的進程，包括其他使用者的。
- -u：顯示詳細的使用者信息，如使用者名稱、CPU 使用率等。
- -x：顯示沒有控制終端的進程。
- -e：顯示所有進程，等同於 -A 選項。
- -f：顯示完整的格式，包括父進程和其他資訊。
- -l：以長格式顯示詳細資訊，包括部分資源使用情況。

|進程類型|控制終端|說明|
|---|---|---|
|在 terminal 裡手動啟動的程式（如 vim、python）|有|能接收 Ctrl+C|
|使用 SSH 登入後執行的程式|有|控制終端為 SSH 分配的 pseudo-TTY|
|系統 daemon（如 nginx, sshd）|**沒有**|它們是背景服務，不跟終端互動|
|使用 `nohup`, `&`, `cron`, `systemd` 啟動的服務|**沒有**|完全脫離 TTY|

---



