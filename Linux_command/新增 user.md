## Step 1: 新增用戶

```bash
sudo useradd -m -s /bin/bash alice
```
- `-m`: 附加參數，代表 make home directory 。意思是在 `/home` 下建立家目錄 (`/home/user`)
- `-s /bin/bash`: 設定登入後的預設「Shell（終端機介面）」。
## Step 2-a: 設定初始密碼(可用 ssh)

```bash
sudo passwd alice
```

## Step 2-b: 設定 ssh 登入

切換到該使用者

```bash
sudo su - alice
```

建立 `.ssh` 資料夾並設定正確權限

```bash
mkdir -p ~/.ssh 
chmod 700 ~/.ssh
```

寫入公鑰到授權名單，並設定權限

```bash
nano ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

退出該使用者身分，回到你原本的帳號

```bash
exit
```
## Step 3: (選擇) 賦予管理員 sudo 權限

```bash
sudo usermod -aG sudo alice
```

## 確認該使用者是否有 sudo 權限

```bash
sudo -l -U username
groups username
```

### 從 sudo 群組移除

```bash
# Ubuntu/Debian（sudo 群組）
sudo gpasswd -d 使用者名稱 sudo

# CentOS/RHEL（wheel 群組）
sudo gpasswd -d 使用者名稱 wheel
```
### 有獨立的 sudoers 設定檔

```bash
# 檢查是否有單獨設定
sudo ls /etc/sudoers.d/

# 若有對應檔案，直接刪除
sudo rm /etc/sudoers.d/使用者名稱
```

---
## 查詢所有 user 跟 group

```shell
getent passwd | cut -d: -f1
getent group | cut -d: -f1
```

>[!question] 如何實作出查詢所有 user 跟 group 的 linux kernel 程式?

## 刪除 user 跟 group

```bash
# 確認使用者是否登入中 
who | grep username

sudo userdel [username]
sudo userdel -r [username]
```

>[!question] 可以直接把 user 刪掉嗎?
>1. 確認該使用者是否還有正在執行的 Process
>2. 處理「孤兒檔案 (Orphaned Files)」
>3. 備份機制 `sudo tar -czvf /backup/username_backup.tar.gz /home/[username]`

## 確認刪除

```bash
# 確認已刪除
id username

# 刪除殘留的 cron jobs
sudo crontab -r -u username

# 搜尋系統中仍屬於該 UID 的檔案
sudo find / -user username 2>/dev/null

# 從群組中移除（若未自動移除） 
sudo gpasswd -d username groupname
```

