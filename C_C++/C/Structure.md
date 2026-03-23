## 參考自[C语言中的结构体（struct）详解](https://blog.csdn.net/edward_zcl/article/details/112106703)
# 動機
一般想描述的資料結構常是抽象的，他內部難由單一的變數型態(int、char等)來描述，常常是有多種不同的變數型態或是多個單一變數型態來表達，想想看書籍資料，所以當我們想描述較為抽象的資料結構時，不得不把多個變數綑成一包，好方便調用與管理。
# 語法
## 聲明
```C
struct Student{ //声明结构体 
	char name[20]; //姓名 
	int num; //学号 
	float score; //成绩 
};
struct Student stu1; //定义结构体变量

struct Student{ 
	char name[20]; 
	int num; 
	float score; 
}stu1; //在定义之后跟变量名
```
**注意以上兩個程式在幹同一件事，都是聲明後並給一個該型別變量。**
```C
typedef struct Student{
	char name[20];
	int num;
	float score;
} stu;
struct Student a;  // OK
stu b;             // OK，更簡短

struct Student{ 
	char name[20]; 
	int num; 
	float score; 
}stu;
```
# 🧾 兩段的差異總結

| 功能                  | 第一段 typedef | 第二段 非 typedef       |
| ------------------- | ----------- | ------------------- |
| 定義 `struct Student` | ✔️          | ✔️                  |
| 建立型別別名 `stu`        | ✔️          | ❌                   |
| 宣告變數 `stu`          | ❌           | ✔️（宣告了一個 struct 變數） |
| 能否寫 `stu x;`        | ✔️          | ❌（stu 不是型別）         |
## 初始化
```C
// 方法 1:
struct Student stu1, stu2; //定义结构体变量 
strcpy(stu1.name, "Jack"); 
stu1.num = 18; 
stu1.score = 90.5;

// 方法 2:
stu2 = (struct Student){"Tom", 15, 88.0};
```
細節 :
1. 能直接给[[数组名]]赋值，因为数组名是一个常量
2. 因为数组赋值也是使用{}，方法2不转换的话系统无法区分
# 儲存細節
儲存大小會因為alignment的關係，而比實際的所需來的多。譬如說你做4 byte alignment，那存儲大小必為4的倍數，需要注意的是**對齊是看數據類型本身**。Structure內的排列順序，即為儲存順序，各個變數的對齊位置依其規定。
![[Pasted image 20251114122238.png]]
## 沒有pragma pack
```C
struct A{
	int a;
	char b;
	short c;
}str1;
```
![[Pasted image 20251114122334.png]]
```C
struct A{
	char a;
	int b;
	short c;
} str1;
```
![[Pasted image 20251114122524.png]]
```C
struct A{
	char A[5];
	char B[4];
	int c;
}str1;
```
#### Q: "CPU 讀取資料時，如果資料的起始位址是 4 的倍數（或 8 的倍數等），效率最高。"這句話跟"對齊是跟數據類型本身有關"衝突嗎?
#### A: **因為** CPU (硬體) 處理 4 或 8 位元組倍數的地址最高效 (敘述一)，**所以** 編譯器 (軟體) 規定 `int` (4位元組) 或 `double` (8位元組) 這些「數據類型」必須被放置在 4 或 8 的倍數地址上 (敘述二)。
## 有 pragma pack
`#pragma pack(n)` 让编译器按照n字节对齐。
```C
int main(int argc, const char *argv[]){
#pragma pack(2)
	struct A{
		char a;
		int b;
		short c;
	}str1;
	return 0;
}
```
![[Pasted image 20251114124247.png]]

## struct vs. 函數區域變數
| **特性**    | **struct 成員 (例如 struct { int a; char c; })** | **函式區域變數 (例如 main() { int a; char c; })** |
| --------- | -------------------------------------------- | ----------------------------------------- |
| **保證順序？** | **是** (標準保證 `a` 的位址在 `c` 之前)                 | **否** (編譯器可以任意安排 `a` 和 `c` 的相對位置)         |
| **保證連續？** | **否** (為了對齊，`a` 和 `c` 之間可能有 Padding)         | **否** (編譯器可以任意插入 Padding 或重排)             |
| **主要考量**  | 資料佈局的穩定性、可預測性                                | 執行效能、CPU 優化                               |
| **存在位置**  | 記憶體 (作為一個連續的區塊)                              | 優先在 **CPU 暫存器**，其次在 **堆疊**                |
`struct` 的成員順序和對齊方式**必須是可預測的**，因為程式設計師經常需要將 `struct` 視為一個**單一的、連續的記憶體區塊**來進行操作。
### 例子(`memcpy` (記憶體複製)) :
```c
#include <string.h> // for memcpy

struct LargeData {
    double values[100]; // 800 bytes
    int meta_id;        // 4 bytes
    char description[50]; // 50 bytes
    // ... 還有很多其他成員 ...
};

struct LargeData original_data;
// (假設 original_data 已經被填滿了資料)

struct LargeData backup_data;

// 手動複製 (非常繁瑣且容易出錯)：
// backup_data.meta_id = original_data.meta_id;
// strcpy(backup_data.description, original_data.description);
// for (int i = 0; i < 100; i++) {
//     backup_data.values[i] = original_data.values[i];
// }
// ... 如果 struct 增加成員，這裡就會忘記 ...

// 使用 memcpy (高效且保證完整)
memcpy(&backup_data, &original_data, sizeof(struct LargeData));
```
只有因為`struct` 的成員順序和對齊方式**必須是可預測的**，才可以直接memcpy。
# 結構體數組
## 結構數組定義
```C
struct Student{ //声明结构体 Student 
	char name[20]; 
	int num; 
	float score; 
}stu[5]; //定义一个结构结构数组stu，共有5个元素
```
## 初始化
```C
// 定義好後初始化
struct Student stu[2] = {{"Mike", 27, 91},{"Tom", 15, 88.0}};
```
## 例子
```C
#include <stdio.h>

struct Student{ //声明结构体 Student 
	char name[20] = "ji"; 
	int num = 0; 
	float score = 0.1; 
}stu[5]; //定义一个结构结构数组stu，共有5个元素

int main(void) {

	for(int i=0; i<sizeof(stu)/sizeof(struct Student); ++i){
		printf("name: %s, num: %d, score: %f\n", stu[i].name, stu[i].num, stu[i].score);
	}

	return 0;

}
```
==**這程式是錯的**== ，struct內只是給藍圖不能賦值。**正確程式**如下:
```C
#include <stdio.h>

struct Student{ //声明结构体 Student 
	char name[20]; 
	int num; 
	float score; 
}stu[5]; //定义一个结构结构数组stu，共有5个元素

int main(void) {

	for(int i=0; i<sizeof(stu)/sizeof(struct Student); ++i){
		printf("name: %s, num: %d, score: %f\n", stu[i].name, stu[i].num, stu[i].score);
	}

	return 0;

}
```
這程式的變數尚未初始化，所以值由在哪定義決定
1. 如果變量是在 **全域（Global）或靜態（Static）** 區宣告 : **初始值保證為 0。** **原因：** 這些變量儲存在 **資料段（Data Segment）** 或 **BSS 段**，它們會在程式載入時被作業系統或程式啟動代碼自動初始化。細想定義在global或static的資料是較少的且會被大量的函示跟程式存取，所以可以費時去整理他，也需要初始化為0避免歧異。
2. 如果變量是在 **區域（Local）** 區宣告（即在函式內部) : **初始值：** **未定義**。**原因：** 這些變量儲存在 **堆疊（Stack）** 上。為了追求最高的執行效率，C 語言規定編譯器不會浪費時間去清空或初始化堆疊上剛分配的記憶體。

# 結構和指針
## 定義指針
```C
struct Student *pstu;       //定义了一个指针变量，它只能指向Student结构体类型的结构体变量
```
## 結構體嵌套
```C
// 嵌套別人
struct Birthday{                //声明结构体 Birthday
    int year;
    int month;
    int day;
};
struct Student{                 //声明结构体 Student
    char name[20];              
    int num;                    
    float score;                 
    struct Birthday birthday;   //生日
}; 

// 嵌套自己
struct Student{                 //声明结构体 Student
    char name[20];
    int num;
    float score;
    struct Student *friend;     //嵌套定义自己的指针
}

// 多層嵌套
struct Time{                    //声明结构体 Time
    int hh;                     //时
    int mm;                     //分
    int ss;                     //秒
};
struct Birthday{                //声明结构体 Birthday
    int year;
    int month;
    int day;
    struct Time dateTime        //嵌套结构
};
struct Student{                 //声明结构体 Student
    char name[20];
    int num;
    float score;
    struct Birthday birthday;   //嵌套结构
}
//定义并初始化
struct Student stud = {"Jack", 32, 85, {1990, 12, 3, {12, 43, 23}}};
//访问嵌套结构的成员并输出
printf("%s 的出生时刻：%d时 \n", stud.name, stud.birthday.dateTime.hh);
//输出结果：Jack 的出生时刻：12时 
```