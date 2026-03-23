# 進入 gdb
## Step 1: 編譯程式並啟用除錯資訊

```gdb
gcc -g -o myprog myprog.c
```

參數說明: 
1. `-o`
	- 說明  
	    制定目標名稱，缺省的時候，`gcc`編譯出來的文件是`a.out`
2. `-g`
	 只是編譯器，在編譯的時候，產生調試信息
3. 優化參數(bonus)
	-O0
	-O1
	-O2
	-O3
		編譯器的優化選項的4個級別，`-O0`表示沒有優化，`-O1`為缺省值，`-O3`優化級別最高

## Step 2: 啟動 GDB

```gdb
gdb myprog
```
## Step 3: 設定斷點並執行

```gdb
break main
```
如果要特定行，可以用
```gdb
break <line>
```
全部的斷點可以用
```gdb
info breakpoints
```

```gdb
run
```

---
# 常用指令
以下面程式做示範
```c
int main(){
	int a[3];
	struct { double v[3]; double length; } b[17];
	int calendar[12][31];
}
```

1. `print` 或 `p`
2. `whatis`
3. `list`
4. `x/4 b`