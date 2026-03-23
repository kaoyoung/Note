## Step 1: Create a Non-Root User
1. Log in to your server as root: `ssh root@your_server_ip`
2. Create a new user (replace `username` with your desired name): `adduser username`
3. Add the new user to the `sudo` group so they can run administrative commands: `usermod -aG sudo username`

>[!note] adduser command:
>添加新用戶時，會在 `/home` 目錄下創建新用戶目錄。
>語法: `sudo adduser [option] user`
>- option: 
>	- `--system` : 添加系統用戶或組
>	- `--disabled-login` : 禁止登入到用戶帳戶
>	- `--disabled-password` : 禁止使用密碼登入

>[!note] usermod command:
>免去到 `/etc/passwd` 或 `/etc/shadow` 改資料的麻煩，可直接修改用戶相關的設定。
>語法: `usermod [option] username`
>- option:
>	- `-G` : 後面接附加群組(supplementary group)，修改這個使用者能夠支援的群組，修改的是 `/etc/group` 
>	- `-a` (append) : 與 `-G` 合用，可『**增加**次要群組的支援』而非『設定』喔！
>	- `-g` : 後面接主要群組(primary group)，修改 /etc/passwd 的第四個欄位，亦即是 GID 的欄位！
>
>所以 `usermod -aG sudo username`  代表:「把某個使用者加入管理員群組，讓他擁有系統的最高權限」。
>`-g` 跟 `-G` 差在哪 ? 這問題等價於附加群組跟主要群組差在哪 ?
>- 主要群組: 每個使用者**只能有 1 個**主要群組。當你建立一個新檔案或資料夾時，系統預設會把這個檔案的「群組擁有權」歸屬於你的主要群組。
>- 附加群組: 一個使用者可以加入**多個**附加群組。這通常是用來賦予使用者額外的特定權限（例如：加入 `sudo` 群組獲得管理員權限，或加入 `docker` 群組來操作容器）。
>- 注意: **新增用戶到其他附加群組一定要加 `-a` 表示新增，不然會直接修改該用戶附加群組名單。**
## Step 2: Set Up Key Authentication
1. **On your local computer** (not the server), open a terminal and generate an SSH key pair: `ssh-keygen -t ed25519 -C "your_email@example.com"`
2. - Copy the public key to your new server.
    - **On Mac/Linux:** `ssh-copy-id username@your_server_ip`        
    - **On Windows (PowerShell):** `cat ~/.ssh/id_ed25519.pub | ssh username@your_server_ip "mkdir -p ~/.ssh && chmod 700 ~/.ssh && touch ~/.ssh/authorized_keys && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys`        
3. Test the login. Open a new terminal window and try to SSH in as your new user: `ssh username@your_server_ip`. It should log you in without asking for a traditional password.

>[!note] ssh-keygen command
>語法: `ssh-keygen -t ed25519-C "azureuser@myserver"`
>- `-t` (type) : 要建立的金鑰類型
>- `-C` (comment) : 附加至公開金鑰檔案結尾以便輕鬆識別的註解

>[!note] ssh-copy-id command
>語法: `ssh-copy-id -i ~/.ssh/my_custom_key.pub user@192.168.1.100`
>- `-i`(identity file) : 指定要傳送哪一個公鑰檔案
>為何上面的方法沒指定 identity file，因為 default identity is your "standard" ssh key。公鑰檔案包含 public key 跟 private key ，default identity 常在 `~/.ssh` directory, normally named `identity`, `id_rsa`, `id_dsa`, `id_ecdsa` or `id_ed25519`。
>使用這方法要注意私鑰檔案的權限，如果私鑰檔案的權限太鬆，ssh會認為它不安全而拒絕連線，本地端的私鑰權限鑰設為600 ， `.ssh` 目錄權限鑰設為700；伺服器端的`authorized_key` 檔案權限鑰設為 600，`.ssh` 目錄權限鑰設為700。
## Step 3: Harden the SSH Configuration
1. Open the SSH configuration file using your new user account: `sudo nano /etc/ssh/sshd_config`    
2. Find the following lines, uncomment them (remove the `#`), and change them to look exactly like this:    
    - `PermitRootLogin no` _(Stops attackers from trying to brute-force the "root" account)_
    - `PasswordAuthentication no` _(Forces everyone to use SSH keys)_
    - `Port 2222` _(Optional but recommended: Change the default port from 22 to a random high number like 2222, 4567, etc. This stops 99% of automated scanner bots)._
3. Save the file (Press **Ctrl+O**, **Enter**, then **Ctrl+X**).
4. Restart the SSH service to apply the changes: `sudo systemctl restart ssh`

> **Critical Note:** Keep your current SSH session open while you **test logging in through a _new_ terminal window**. If you made a mistake and get locked out, your active session will let you fix it!
## Step 4: Configure a Software Firewall (UFW)
A firewall acts as a bouncer, only allowing traffic on specific ports that you explicitly authorize.
1. Install Uncomplicated Firewall: `sudo apt install ufw`
2. Deny all incoming traffic by default, but allow outgoing: 
		`sudo ufw default deny incoming`
		`sudo ufw default allow outgoing`
3. Allow SSH connections. **Important:** If you changed your SSH port in the previous step to 2222, use that number here: 
	`sudo ufw allow 2222/tcp`
4. Allow HTTP and HTTPS (if you are hosting a website): 
	`sudo ufw allow 80/tcp` 
	`sudo ufw allow 443/tcp`
5. Enable the firewall: `sudo ufw enable`

>[!warning]
>If you haven't run the `allow` command for your specific SSH port yet, **do not close your current terminal window.** If you do, the "Established" session ends, and the "Default Deny" rule will prevent you from ever getting back in without physical access to the machine or a recovery console.

## Step 5: Install Fail2Ban
1. 安裝： `sudo apt install fail2ban`
2. 建立自定義設定檔（不要直接改預設檔）：
	`sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
3. 編輯設定：
	`sudo nano /etc/fail2ban/jail.local` 找到 `[sshd]` 區塊，確保它看起來像這樣（如果你改了端口，記得修改 `port`）：
```text
    [sshd]
    enabled = true
    port    = 2222
    logpath = %(sshd_log)s
    backend = %(sshd_backend)s
    maxretry = 5
	findtime  = 10m
    bantime = 1h
```  
4. 重啟服務： `sudo systemctl restart fail2ban`
5. 看狀態: `sudo fail2ban-client status sshd`