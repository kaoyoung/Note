## 參考自菜鳥教程的[Linux cp 命令](https://www.runoob.com/linux/linux-comm-cp.html)
# 基本語法
>cp [options] source dest
>或
>cp [选项] 源文件 目标文件

## 注意以下操作
> -r : 這是給==目錄==用的
> -i :  確保目的地有==同名檔==時會跳提醒
---
# 實例
**1. 复制文件到目标目录**

> cp file.txt /path/to/destination/

将 file.txt 复制到 /path/to/destination/ 目录中。

**2. 复制文件并重命名**

> cp file.txt /path/to/destination/newfile.txt

将 file.txt 复制到 /path/to/destination/ 目录并重命名为 newfile.txt。

**3. 递归复制目录**

> cp -r /path/to/source_dir /path/to/destination/

将 source_dir 目录及其内容递归复制到 destination 目录。

**4. 交互模式复制**

> cp -i file.txt /path/to/destination/

如果目标位置已存在同名文件，会提示用户确认是否覆盖。

**5. 保留文件属性**

> cp -p file.txt /path/to/destination/

复制文件并保留其原始属性（如权限、时间戳等）。

**6. 仅复制更新的文件**

> cp -u file.txt /path/to/destination/

仅当 file.txt 比目标文件新时才复制。

**7. 显示复制过程**

> cp -v file.txt /path/to/destination/

显示复制的详细信息。

**8. 创建硬链接或符号链接**

>cp -l file.txt /path/to/destination/  # 创建硬链接
>cp -s file.txt /path/to/destination/  # 创建符号链接

**9. 复制多个文件到目录**

> cp file1.txt file2.txt /path/to/destination/

将多个文件复制到目标目录。

**10. 使用通配符复制**

>cp *.txt /path/to/destination/

复制所有 .txt 文件到目标目录。

**11. 结合 find 命令复制特定文件**

>find /path/to/source -name "\*.log" -exec cp {} /path/to/destination/  \\;

查找并复制所有 .log 文件到目标目录。

---
# 注意事項
1. 如果目标路径是一个目录，`cp` 会将源文件或目录复制到该目录中。
2. 如果目标路径是一个文件名，`cp` 会将源文件复制并重命名为目标文件名。
3. ==复制目录时，必须使用 `-r` 或 `-R` 选项，否则会报错。==
4. 如果目标文件已存在，==默认情况下 `cp` 会覆盖它（除非使用 `-i` 选项）。==
---
# 問題
## Q1. 已經有`ln`了為何要`cp -l`?
首先`ln`有一個痛點無法對目錄做硬連接(這是linux規定，怕無窮迴圈)，他對於目錄只有這招
1. 先去 `dest` 建立對應的空目錄（因為不能 Link 目錄）。    
2. 進入該目錄。    
3. 對該目錄下的檔案一個一個執行 `ln`。    
4. 遇到子目錄，重複步驟 1...
==非常麻煩==。
`cp -rl /src /dest` 解決這麻煩 : 自動化的「目錄重建 + 檔案連結」
- **遇到目錄** $\rightarrow$ `mkdir` (建立新 Inode，合法)
- **遇到檔案** $\rightarrow$ `ln` (建立 Hard Link，省空間)

**使用場景**
- **結論：** `cp -l` 無法讓「目錄本身」變成 Hard Link（這依然是被 Kernel 禁止的），但它能讓你用極少的空間「複製」整棵目錄樹的內容。    
- **主要用途：** 這是 Linux 上做 **「快照式備份」(Snapshot Backup)** 的核心技巧。
    - 例如：你昨天的備份在 `/backup/day1`。        
    - 今天要備份時，先執行 `cp -rl /backup/day1 /backup/day2`。        
    - 這瞬間完成，且幾乎不佔硬碟空間（因為全是 Hard Link）。        
    - 接著你再用 `rsync` 僅更新 `/backup/day2` 裡面有變動的檔案。

**C++對應linux觀念**

|**Linux 概念**|**C++ 對應概念**|**原理說明**|
|---|---|---|
|**Inode** (檔案內容)|`new Object()` (Heap 上的物件)|實際存放資料的地方。|
|**Hard Link** (檔名)|`std::shared_ptr<T>`|擁有所有權的指標。**具有引用計數 (Reference Counting)**。|
|**rm 指令** (刪除)|`sp.reset()` 或超出作用域|減少引用計數。計數歸零時，才真正釋放記憶體/磁碟空間。|
|**Symbolic Link** (軟連結)|`std::weak_ptr<T>` 或 `Raw Pointer`|**不增加引用計數**。如果本尊死了，它就指不到東西 (Broken Link)。|
**總結 cp -rl 在搞啥**
利用目錄檔案常比文件檔案小很多的特性加上hard link直接連上inode不會消耗記憶體，來快速複製一整個子目錄樹的結構

> Q : 回顧為何linux規定無法對目錄做硬連接?
> A : 作業系統禁止目錄硬連結的核心原因，並不僅僅是「怕遍歷程式陷入無窮迴圈」(可用快慢指針、Max Depth/TTL、或用inode table紀訪問過的inode解)，而是因為它**破壞了檔案系統的「樹狀結構」與「父子關係的唯一性」，導致更底層的邏輯崩潰**。

**父子關係的唯一性** : 
假設有三個目錄/root/A、/root/B、/root/A/LB現在你`ln -d /root/B /root/A/LB`，問題來了父目錄(..)是誰，/root還是/root/A
**檔案系統的樹狀結構** : 
對於目錄 : 要儲存檔名列表，在POSIX中規定要有..指向父目錄。
對於檔案 : 只有data block，只在乎數據不在乎存在哪，沒有父節點概念。
- **檔案硬連結** $\to$ 匯聚到一點，然後停止 $\to$ **安全**。    
- **目錄硬連結** $\to$ 可以穿過去再繞回來 $\to$ **危險 (無限迴圈)**。
**底層崩潰(Disk Space Leak、Orphaned Inodes) :**
有四個目錄/home/A、/home/B、/home/A/link_B、/home/B/link_A現在`ln -d /root/A/link_B /root/B`和`ln -d /root/B/link_A /root/A` ，然後`rmdir A` 和 `rmdir B`後，這兩檔案不會被刪除因為inode的記數還不為0，但你已經找不到他們了。這造成Orphaned Inodes。
**總結 :**
檔案和目錄最本質的差別一方面為檔案沒必要考慮父親，不用考慮父子結構的關係；另一個是檔案是葉節點他下面肯定沒人，所以不存在他被刪掉，但下面有人導致有孤兒出現。**「檔案是資料的容器，而目錄是結構的樞紐。」**
- **刪除容器**：只是丟掉內容物，乾淨俐落。    
- **刪除樞紐**：如果不小心處理（例如有循環連結），就會導致結構崩塌或部分區域與主幹斷裂（Memory/Disk Leak）。

|**操作**|**Hard Link (ln)**|**Soft Link (ln -s)**|
|---|---|---|
|**針對目錄建立連結**|**禁止** (防止結構崩潰)|**允許** (這是捷徑)|
|**針對檔案建立連結**|**允許** (產生另一個檔名)|**允許** (產生捷徑)|
|**跨型別偽裝**|**不可能** (Inode 決定型別)|**不適用** (連結本身是獨立型別)|






