# 觀念修正

C99 規格書 6.2.5 Types (型別) - 第 20 段
```text
A pointer type describes an object whose value provides a reference to an entity of the referenced type.
```
>這邊的 reference 指的是我們俗稱的「記憶體位置」。 

C99 規格書 6.3.2.1 - 第 2 段
```text
Except when it is the operand of the sizeof operator, the unary & operator, the ++ operator, the -- operator, or the left operand of the . operator or an assignment operator, an lvalue that does not have array type is converted to the value stored in the designated object (and is no longer an lvalue).
```
>"lvalue that does not have array type is converted to the value stored in the designated object" 這段話告訴我們，除非 `lvalue` 是 array type ，不然我們要的是 object 內的值

>[!note]
>對於 `char a = 'z'; char *b = &a; char *c = b` 中的 `char *c = b` 我們是把 `b` 內儲存的值(地址) 讀出來，再賦值給 `c`。我們可以將**指標視為變數的一種，裡面存的是位置**。

---
# extern 用途
C99 規格書 6.7.1- 第 1 段
```text
storage-class-specifier: 
	typedef 
	extern 
	static 
	auto 
	register
```
- 這告訴我們 `extern` 是 storage-class-specifier

在往下推進前，我們需要理解 c 的 linkage 在說啥。
C99 規格書 6.2.2- 第 1 段
```text
An identifier declared in different scopes or in the same scope more than once can be made to refer to the same object or function by a process called linkage. There are three kinds of linkage: external, internal, and none.
```
先說一下名詞解釋
- identifier (6.2.1/4) : An identifier can denote an object; a function; a tag or a member of a structure, union, or enumeration; a typedef name; a label name; a macro name; or a macro parameter. The same identifier can denote different entities at different points in the program.
- object (3.14) :  region of data storage in the execution environment, the contents of which can represent values
這段文字說了
- linkage : 在不同或相同 scope 的 identifier 可以依靠 **linkage** 參考到同一個物件或程式。
- linkage 分為三種 : external 、internal 、none。
C99 規格書 6.2.2- 第 2 段
```text
In the set of translation units and libraries that constitutes an entire program, each declaration of a particular identifier with external linkage denotes the same object or function. Within one translation unit, each declaration of an identifier with internal linkage denotes the same object or function. Each declaration of an identifier with no linkage denotes a unique entity.
```
- 不同編譯單元間 : external linkage；同一個編譯單元 : internal linkage；沒任何關係 : none

最後看一下 `extern` 到底在幹嘛
C99 規格書 6.2.2- 第 4 段
```text
For an identifier declared with the storage-class specifier extern in a scope in which a prior declaration of that identifier is visible, if the prior declaration specifies internal or external linkage, the linkage of the identifier at the later declaration is the same as the linkage specified at the prior declaration. If no prior declaration is visible, or if the prior declaration specifies no linkage, then the identifier has external linkage.
```
- 這說明如果 `extern` 前面有 internal linkage (例如: `static`)那麼 `extern` 就不會是 external linkage 而是 external linkage
- 先寫 `static int x = 10; extern int x` 這是**合法**的，但 `extern int x; static int x` 這是**非法**的。

C99 規格書 6.2.2- 第 5 段
```text
If the declaration of an identifier for a function has no storage-class specifier, its linkage is determined exactly as if it were declared with the storage-class specifier extern. If the declaration of an identifier for an object has file scope and no storage-class specifier, its linkage is external
```
先說一下名詞解釋
- file scope (6.2.1/4) : If the declarator or type specifier that declares the identifier appears **outside of any block or list of parameters**, the identifier has file scope, which terminates at the end of the translation unit.
這段文字說明
- function 就算不加 extern，也默認具有 extern 的特性。
- 如果一個識別符 (identifier) 是 file scope 且沒聲明 storage-class 那它自動是 external linkage。
>[!note] external linkage 跟 `extern` 區別
>假設在 `file1.c` 中寫 `extern int x;`，在 `file2.c` 中的 file scope 寫 `int x;` 這是**合法**的，但在 `file1.c` 中寫 `int x;`，在 `file2.c` 中的 file scope 寫 `int x;` 這是**非法**的。你可能疑惑依C99 規格書 6.2.2- 第 5 段的說明，`int x` 是 external linkage 那它可以去地方找阿，怎麼會 multiple definition，這是因為 C 語言的 tentative definition (如果在 file scope 宣告一個物件的識別碼，沒有初始值，且沒有儲存類別指定符號（或使用 `static`），這就構成了一個**暫定定義**)，如果一個編譯單元內包含了一個或多個暫定定義，且到檔案結尾都沒有出現真正的外部定義（例如 `int x = 10;`），那麼編譯器**會自動把它當作一個初始化為 0 的外部定義 (`int x = 0;`)**，所以對同一個 identifier 有兩個外部定義，造成 multiple definition。注意 `extern int x;` 只是個宣告 (declaration)。
>

---
# 例子
1. 
在 file1.c 中寫
```c
char a[] = "ABCDE";
```
在 file2.c 中寫
```c
extern char *a;

void print_str(){
	printf("%s", a);
}
```
會發生甚麼事。
>[!answer]
>會報錯 (Segmentation fault (core dumped)) 。在 `file1.c` 中寫 `char a[] = "ABCDE";` 我們可以假設在 `0x1000` 存 `'A'`、在 `0x1001` 存 `'B'`、在 `0x1002` 存 `'C'`、在 `0x1003` 存 `'D'`、在 `0x1004` 存 `'E'`、在 `0x1005` 存 `\0`。這時在 `file2.c` 中寫 `printf("%s", a);` ，會依照 `extern char *a;` 知道 `a` 是一個指標而我們需要去讀指標內的值來知道欲輸出字串的地址 (看觀念修正，**指標視為變數的一種，重要的是裡面的值(地址)**)，這時跑到 `0x1000` 讀值，得到值(地址)為 `0x44434241` (假設Little Endian ，且為32位元系統)，接著去 `0x44434241` 找欲輸出字串，發現那裡根本是作業系統的禁區（或根本不存在這塊記憶體），Segmentation fault！
>

須注意以下程式是好的
```c
#include <stdio.h>

int main(){
	char a[] = "ASDFG";
	char *b = a;
	printf("%s", b);
}
```