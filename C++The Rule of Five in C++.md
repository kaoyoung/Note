## Ref: [The Rule of Five in C++](https://www.geeksforgeeks.org/cpp/rule-of-five-in-cpp/)

# 為何需要這規範

>[!note] 觀念一
>This ensures proper copying, moving, and destruction of objects that manage resources
>- Helps manage dynamically allocated resources safely.
>- Prevents memory leaks, double deletion, and shallow-copy issues.

只要一個 class 的 member function 有用到資源管理相關的操作時，那該 class 必須要支援 destructor, copy constructor, copy assignment operator, move constructor, move assignment operator 這五個功能已避免 memory leak, double deletion, and shallow-copy 的問題。

>[!question] 問題一
>試描述 memory leak, double deletion, shallow-copy 可能導致的問題

這幾個問題都關於記憶體。先說一下 memory leak (記憶體洩漏)，如果你向記憶體要了一段記憶體空間，在使用之後沒有還回去，該記憶體空間便會被一直占用，如下例子
```C
// char name[10] = malloc(10*sizeof(char));
char name[10] = malloc(10*sizeof(char));

// if you forget to type the following code, after using this char array, it will cause memory leakage
// free(name)
```

>[!Note] `char name[10] = malloc(10*sizeof(char));` 這寫法是錯的
>`malloc()` 會回傳一個 pointer 而 `char name[10]` 是一個 array 且這時 `char name[10]` 不會 decay 成 pointer，所以會報錯(error: invalid initializer)。對於何時 array 會 decay 成 pointer 可以參考這一篇 [[C Programming FAQs (Section 6 Array and Pointer)]]。

double deletion 的發生主要原因在於記憶體空間會重複使用，假如重複釋放一個物件，那該記憶體位置的值會被清空兩次，而在第一次清空記憶體位置時，該位置會被放入 free list 的清單供 OS 來調度，之後第二次刪除時，該記憶體位置可能已經分配給別人，所以有可能刪除到別人的資料，如下例子
```C
char name[10];
char *copy_name = name;

free(name)
free(copy_name)
```
shallow copy 是指不完全複製一個物件單純依靠複製一份該記憶體位置的方式達成複製的目的，這樣做確實複製出一份找到該物件的途徑，但這時對該物件的修改會影響到其他所有持有該物件 shallow copy 的其他人，如下例子
```C
char name[10];
char *copy_name = name;

copy_name[0] = 'A';
name[0] = 'B';
```
明顯 `name` 的第一個字元會被改成 `B`。

>[!question] 問題一之一
>細說一下 OS 對空閒記憶體的調度，如何影響 double deletion ? 如何避免 double deletion ?

要回答這問題需要先理解 `malloc` 跟 `free` 這兩 function call 在做啥，接下來的資料參考自 [實作與解析 malloc & free](https://nicknick0630.github.io/2019/05/13/%E5%AF%A6%E4%BD%9C%E8%88%87%E8%A7%A3%E6%9E%90-malloc-free/)。一個經典的 process 分布圖如下
![[Pasted image 20260905232530.png]]

一個要新記憶體的方式是更改 heap 頂端的位置，在 linux 中提供兩個 system call
```C
#include <unistd.h>

int brk(void *addr); // brk() 會把 program break 設定到 addr 如果設定成公會 return 0，失敗會回傳 -1

void* sbrk(intptr_t increment); // sbrk() 會把 program break 增加 increment，設定成功就會回傳上一個 program break 位址，失敗會回傳 (void*)-1
```
**program break**: It defines the end of  the process’s data segment (i.e., the program break is the first location after the end of the uninitialized data segment). Increasing the program break has the effect of allocating memory to the process; decreasing the break deallocates memory.
除了用 `brk, sbrk` 外還可以使用 `mmap, munmap` 來調整 heap 所使用的記憶體空間。
在理解如何增加 heap 空間後，解釋一下 free list 的設計思路
![[Pasted image 20260906010816.png]]
由上圖知道 free list 透過 linked list 的結構儲存空閒的記憶體，這些記憶體構成一個 memory pool 讓新的記憶體請求 (`malloc()`) 不用透過 `brk(), sbrk()` 來繼續索取，而是可以先試著在這 memory pool 找，這邊找的邏輯有 best-fit, first-fit 等思路，當然這 free list 還可以先分類好 memory pool 中 block 的大小來加速搜尋，如下圖所示
![[Pasted image 20260906011313.png]]
從上面對於 free list 的描述可以知無法直接在 user space 的 allocator 層面阻止 double free (至少在 free list 的架構)，因為記憶體真的重複再使用。

>[!note] user space 的 allocator 跟 kernel 的關係
>在 `malloc` 時需求的記憶體可能遠小於 kernel 最小的 page size (4 kb)，同時 syscall 太貴 (要 trap 進 kernel 切換 stack 回來)，所以 `malloc, free` 是在 user space 的 allocator 處理，在必要時才用 `brk, mmap` 等手段去 kernel 拿記憶體 (`malloc` 超過 `MMAP_THRESHOLD` 直接 `mmap` 一段獨立區域，`free` 會 `munmap` 還給 kerenl)

在 C 中要避免 double free 的方法基本得依賴寫程式的習慣
- 文件內寫清楚「誰配置、誰釋放」
- 用 `goto` 統一釋放的位置
- `free` 後將指標設為 NULL
C++ 引入了所有權 (ownership) 的概念，讓這問題可以在語法層面直接解決。double deletion 是因為一塊記憶體有多個擁有者，但沒人說清楚誰負責釋放所造成的，C++ 中提供 
- `make_unique<Widget>()`: 唯一擁有者，離開作用域自動釋放
- `make_shared<Widget>()`: 共享擁有，引用計數歸零才釋放
`make_unique<Widget>()` 只能 move 不能 copy，語法層面就避免掉共享的情況；`shared_ptr` 讓智能指標自己在記數歸零時自動釋放。或是直接用 AddressSanitizer 來偵測
```bash
g++ -fsanitize=address -g main.cpp
```

---
# Destructor
It's used for removing/freeing up all the resources that an object has taken througout its lifetime. By destructor, we make sure that any resources taken by objects are properly released before the object is no longer in scope.

```C
class ClassName{
public:
	~ClassName(){
		// Release allocated resources
	}
};
```


---
# Copy Constructor
It is used to make a new object by copying an existing object. Copy constructor is invoked when we use it to pass an object by value or when we make a copy explicitly.

```C
class ClassName{
public:
	ClassName(const ClassName& other){
		// Deep copy resources
	}
};
```


---
# Copy Assignment Operator
It's a special type of function that takes care of assigning the data of one object to another object. It gets called when you use the assignment operator (=) between objects.

```C
class ClassName{
public:
	ClassName& operator=(const ClassName& other){
		if(this != &other){
			// Deep copy resources
		}
		return *this;
	}
};
```


---
# Move Constructor
It's one of the memeber functions that is used to transfer the ownership of resources from one object to another. 

```C
class ClassName{
public:
	ClassName(ClassName&& other) noexcept{
		// Transfer ownership
	}
};
```

---
# Move Assignment Operator
It's used when an existing object is assigned to the value of an rvalue. It is activated when you use the assignment operator (=) to assign the data of a temporary object(value) to an existing object.

```C
class ClassName{
public:
	ClassName& operator=(ClassName&& other) noexcept{
		if(this != &other){
			// Transfer ownership
		}
		return *this;
	}
}
```



---
# Example

```C
#include <iostream>
#include <utility> // for using move
using namespace std;

class ResourceManager {
private:
    int* data;
    size_t size;

public:
    // default constructor
    ResourceManager(size_t sz = 0)
        : data(new int[sz])
        , size(sz)
    {
        cout << "Default Constructor is called" << endl;
    }

    // Destructor
    ~ResourceManager()
    {
        delete[] data;
        cout << "Destructor is called" << endl;
    }

    // Copy Constructor
    ResourceManager(const ResourceManager& other)
        : data(new int[other.size])
        , size(other.size)
    {
        copy(other.data, other.data + other.size, data);
        cout << "Copy Constructor is called" << endl;
    }

    // Copy Assignment Operator
    ResourceManager& operator=(const ResourceManager& other)
    {
        if (this != &other) {
            delete[] data;
            data = new int[other.size];
            size = other.size;
            copy(other.data, other.data + other.size, data);
        }
        cout << "Copy Assignment Operator is called"
             << endl;
        return *this;
    }

    // Move Constructor
    ResourceManager(ResourceManager&& other) noexcept
        : data(other.data),
          size(other.size)
    {
        other.data = nullptr;
        other.size = 0;
        cout << "Move Constructor" << endl;
    }

    // Move Assignment Operator
    ResourceManager&
    operator=(ResourceManager&& other) noexcept
    {
        if (this != &other) {
            delete[] data;
            data = other.data;
            size = other.size;
            other.data = nullptr;
            other.size = 0;
        }
        cout << "Move Assignment Operator" << endl;
        return *this;
    }
};

int main()
{
    // Creating an object using default constructor
    ResourceManager obj1(5);

    // Creating an object using Copy constructor
    ResourceManager obj2 = obj1;

    // Creating an object using Copy assignment operator
    ResourceManager obj3;
    obj3 = obj1;

    // Creating an object using Move constructor
    ResourceManager obj4 = move(obj1);

    // Creating an object using Move assignment operator
    ResourceManager obj5;
    obj5 = move(obj2);

    return 0;
}
```

---
# Advantages and Best Practices
Advantages
- Improves performance by avoiding unnecessary deep copies
- Makes resource management safer and more predictable
Best Practices
- Prefer move operations whenever ownership can be transferred
- Use noexcept for move constructor and move assignment operator
- Consider using smart pointers to avoid manual resource management whenever possible

>[!question] 問題二
>為何 destructor, copy constructor, copy assignemnt operator, move constructor, move assignement operator 做出來後可以確保安全? 這是最少的操作嗎


