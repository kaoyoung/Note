# Reference: [连接带图形界面的 Ubuntu 服务器](https://zhuanlan.zhihu.com/p/679689697)
# Ubuntu GUI
## Install xfce4
```text
# 更新 apt 信息
sudo apt update

# 安装Xfce桌面
sudo apt install xubuntu-desktop
sudo apt install xfce4 xfce4-goodies xorg dbus-x11 x11-xserver-utils

# 设置默认桌面
echo xfce4-session > ~/.xsession
```

- `sudo apt update` : 叫系統去檢查遠端伺服器上有哪些最新的軟體包資訊。安裝任何新軟體前，這是一個好習慣，確保你下載的是最新且相容的版本。
- `sudo apt install xubuntu-desktop` :  安裝 **Xubuntu** 的完整桌面環境。
- `sudo apt install xfce4 xfce4-goodies xorg dbus-x11 x11-xserver-utils`
	- `xfce4`、`xfce4-goodies` : 這將包括額外的主題、圖標、窗口效果和各種增強工具，從而提供更加完整的Xfce體驗。
	- `xorg` : 負責底層繪圖、管理鼠標、鍵盤、顯卡和顯示器，為圖形應用程式（X Client）提供顯示介面
	- `dbus-x11`、`x11-xserver-utils` : 這些是讓視窗系統能正常溝通、管理螢幕解析度與輸入裝置的工具。
- `echo xfce4-session > ~/.xsession` : 在你的家目錄建立一個名為 `.xsession` 的隱藏檔，並寫入 `xfce4-session`
## Install xdrp
```text
# 安装
sudo apt install xrdp
# 允许xrdp 运行 
sudo systemctl enable xrdp

# 需要添加xrdp到 ssl-cert group 
sudo adduser xrdp ssl-cert
```

- `sudo apt install xrdp` : 一個開源的 RDP（Remote Desktop Protocol）伺服器，讓 Linux 能夠理解 Windows 遠端桌面傳過來的訊號。
- `sudo adduser xrdp ssl-cert` :  ssl憑證是網站必備的安全技術，透過加密伺服器與瀏覽器間的傳輸數據（如密碼、信用卡號），防止資訊被竊聽或竄改

# SSH from Window
## Step 1: create tunnel
```text
ssh -L 3389:127.0.0.1:3389 帳號@你的伺服器IP
```

- 3389 port: Windows 遠端桌面協定 (RDP) 的預設連接埠
- `-L` : Local Port forwarding (Local to Remote)，格式是`本地埠:目標主機:目標埠`。
- `127.0.0.1` : 本機地址，主要用於測試。
- 意思 : 所有對你**本地電腦**的 3389 埠的連線請求，都會透過 SSH 通道被加密轉送到你連線的**遠端伺服器**，然後由該伺服器去連接 `127.0.0.1` 的 3389 埠。

## Step 2: 在本地啟用遠端桌面
1. 開啟遠端桌面連線
2. 在遠端桌面連線的電腦欄位輸入 `127.0.0.1` 或 `localhost` 。SSH 指令已經把遠端伺服器的 3389 埠口「拉」到了你本機電腦的 3389 埠口。對遠端桌面軟體來說，它現在以為目標伺服器就在你自己的電腦上。