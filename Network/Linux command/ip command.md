## 可操作對象
- Object : 對象
- link : 網路設備
- address : 定義ipv4 ipv6的地址
- neighbour/neighbor : 查看ARP緩存地址
- route : 路由表對象
- maddress : 多播地址
- tunel :  IP上的地址
## 常用指令
1. 查看，顯示網路設備信息
	- ip addr show
	- ip link show dev ens33
	- ip -s link show dev ens33
2. 關閉、激活網路設備
	- ip link set ens33 down
	- ip link set ens33 up
3. 修改網卡mac地址
	- ip link set ens33 address 0:0c:29:13:10:11
4. 顯示網卡訊息
	- ip a
	- ip addr show
5. ip命令添加，刪除ip信息
	- ip address add 192.168.178.160/24 dev ens33
	- ip address del 192.168.178.161/24 dev ens33
6. ip命令給網卡添加別名
	- ip address add 192.168.78.188/24 dev ens33 label ens33:1
7. 通過命令檢查路由信息
	- ip route
 8. ip檢查ARP緩存、檢查MAC地址信息
	 - ip neighbour
	 - ip neighbor