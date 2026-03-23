### 參考自[聊聊C++ explicit关键字：理解与实践](https://zhuanlan.zhihu.com/p/1913980241547858151)
## explicit在幹嘛
`explicit`关键字用于**禁止构造函数的隐式类型转换**。Compiler 不会自动使用该构造函数进行隐式转换。
## 為什麼需要explicit
1. 避免重载决议歧义错误
```cpp
#include <iostream>

class Cents {
    int cents_;
public:
    Cents(int c) : cents_(c) { std::cout << "Cents: " << c << "\n"; }
};

class Dollar {
    int dollar_;
public:
    Dollar(int c) : dollar_(c) { std::cout << "Dollar: " << c << "\n"; }
};

void payBill(Cents amount) {
    std::cout << "Paying with cents\n";
}

void payBill(Dollar amount) {
    std::cout << "Paying with dollars\n";
}

int main() {
    payBill(500);   // ❌ 编译错误：ambiguous call
    return 0;
}
```
通过使用 explicit，要求必须明确意图的进行调用，修改上面的代码
```cpp
// 1. 构造函数前加explicit关键字
explicit Cents(int c) : cents_(c) { std::cout << "Cents: " << c << "\n"; }  // 禁止隐式转换
explicit Dollar(int c) : dollar_(c) { std::cout << "Dollar: " << c << "\n"; }  // 禁止隐式转换


// 2.明确调用意图
// payBill(500);           // ❌ 此时会报编译错误：没有匹配的重载
                          //  error: no matching function for call to ‘payBill(int)
payBill(Cents(500));    // ✅ 必须显式转换， 明确意图
payBill(Dollar(500));   // ✅ 必须显式转换， 明确意图
```
2. 避免性能陷阱
```cpp
class Matrix {
    double* data;
    size_t size_;
public:
    Matrix(size_t size) : size_(size) {
        data = new double[size * size];  // 昂贵的内存分配！
    }
};

void compute(const Matrix& m);

// 性能陷阱
compute(1000);  // 意外创建了1000x1000的矩阵！
```
通过使用 explicit，要求必须明确意图的进行调用，修改上面的代码
```cpp
// 1. 构造函数前加explicit关键字
explicit Matrix(size_t size) : size_(size) {
    data = new double[size * size]; 
}

// 2.明确调用意图
// compute(1000);           // ❌ 编译错误
compute(Matrix(1000));   // ✅ 程序员明确知道在创建矩阵
```
3. 防止意外的类型转换
```cpp
class FileHandle {
    FILE* file_;
public:
    FileHandle(const char* filename) {
        file_ = fopen(filename, "r");  // 可能失败的系统调用
    }
};

void processFile(FileHandle fh);

// 可能传错参数类型，导致编译错误
processFile("config.txt");  // 看起来像传字符串，实际创建了FileHandle对象
// 可能会发生的事情
// 1. 调用 fopen() - 可能失败
// 2. 创建临时对象
// 3. 传递给函数
// 4. 函数结束后析构（但没有fclose！）
```
通过使用 explicit，要求必须明确意图的进行调用，修改上面的代码，避免可能发生的**资源泄漏风险**
```cpp
// 安全的设计
// 1. 构造函数前加explicit关键字
explicit FileHandle(const char* filename) {
    file_ = fopen(filename, "r");  // 可能失败的系统调用
}

// 2.明确调用意图
// processFile("config.txt");           // ❌ 编译错误，强制程序员明确意图
processFile(FileHandle("config.txt")); // ✅ 意图明确
```
## 核心概念：直接初始化 vs 拷贝初始化
#### 直接初始化（不受explicit影响）
```cpp
class P {
public:
    explicit P(int a, int b, int c);
};

P obj1{1, 2, 3};        // ✅ 直接初始化，总是可以
P obj2(1, 2, 3);        // ✅ 直接初始化，总是可以
P obj2(1.1, 2.2, 3.3);        // ✅ 直接初始化，总是可以
```
#### 拷贝初始化（受explicit影响）
```cpp
P obj3 = {1, 2, 3};     // ❌ 拷贝初始化，explicit会阻止
P obj4 = P{1, 2, 3};    // ✅ 显式构造后拷贝，可以
```
因為拷貝初始化需要用`{1, 2, 3}`创建临时对象，這需要compiler的隱式轉換。
## 拷贝和移动构造函数不能加explicit
[[拷貝和移動函數]]不能加`explicit`，否则会破坏基本语义，破坏函数传递，破坏容器使用，破坏返回值语义。如下：
```cpp
// 永远不要这样做
explicit MyClass(const MyClass& other);  // ❌
explicit MyClass(MyClass&& other);       // ❌

// 1.破坏基本语义
MyClass obj1;
MyClass obj2 = obj1;  // ❌ 如果拷贝构造是explicit，这会编译错误！

// 2.破坏函数传递
void process(MyClass obj);  // 按值传递
MyClass original;
process(original);  // ❌ 如果拷贝构造是explicit，无法传递参数！

// 3.破坏容器使用
std::vector<MyClass> vec;
MyClass obj;
vec.push_back(obj);  // ❌ 容器无法工作！
// 标准库容器依赖于拷贝/移动语义

// 4.破坏返回值语义
MyClass createObject() {
    MyClass obj;
    return obj;  // ❌ 无法返回对象！
}
```
正确的做法
```cpp
class MyClass {
public:
    // ✅ 正常的拷贝和移动构造函数
    MyClass(const MyClass& other) = default;
    MyClass(MyClass&& other) = default;

    // 或者如果不需要拷贝，就删除它们
    MyClass(const MyClass&) = delete;
    MyClass(MyClass&&) = delete;
};
```
### 為什麼拷贝和移动构造函数不能加explicit
1. 破坏基本语义 (Copy Initialization) : `MyClass obj2 = obj1;` 这是**拷贝初始化**。编译器将其视为一种“隐式转换”。
2. 破坏函数传递 (Pass-by-Value) : 当你把对象按值传递给函数时，编译器需要“隐式地”创建一个参数的副本。
3. 破坏容器使用 (STL Compatibility) : 标准库容器（如 `std::vector`, `std::map`）在内部运作时，极度依赖隐式的拷贝和移动行为。当你把对象放入 vector，或者 vector 需要扩容（reallocate）时，它需要把旧元素拷贝或移动到新内存中。
4. 破坏返回值语义 (Return-by-Value) : 当你从函数返回一个对象时，通常会触发隐式构造。
