# Syntax
- Note that there must be ==no spaces around the "`=`" sign==: `VAR=value` works; `VAR = value` doesn't work.
	- In the first case, the shell sees the "`=`" symbol and treats the command as a variable assignment.
	- In the second case, the shell assumes that VAR must be the name of a command and tries to execute it.
- The shell ==does not care about types of variables==; they may store strings, integers, real numbers - anything you like.
- ==Variables in the Bourne shell do not have to be declared==, as they do in languages like C. But if you try to ==read an undeclared variable, the result is the empty string==. You get no warnings or errors.
- There is a command called `export` which has a fundamental effect on the scope of variables.  `export` command can export the variable for it to be inherited by another program
- We can source a script via the "." (dot) command. In order to receive environment changes back from the script, we must ==_source_ the script - this effectively runs the script within our own interactive shell==, instead of spawning another shell to run it.
- unlike most languages, the dollar (`$`) symbol is required when getting the value of a variable, but must not be used when setting the value of the variable.
- The _curly brackets_ can define the variable ends and the rest starts.
# Example
## EX1 :
```shell=
#!/bin/sh  
echo What is your name?  
read MY_NAME
echo "Hello $MY_NAME - hope you're well."
```
>Don't write `echo what's your name`. Otherwise, it will output "Syntax error: Unterminated quoted string"

- This is using the shell-builtin command `read` which reads a line from standard input into the variable supplied.
- What happens, is that ==the `read` command automatically places quotes around its input, so that spaces are treated correctly==. (You will need to quote the output, of course - e.g. `echo "$MY_MESSAGE"`).
### 單引號跟雙引號有啥差 ?
- 雙引號比較「聰明」，它會去解析字串裡面的**變數**或**指令**。
	- **會**解析以 `$` 開頭的變數。    
	- **會**執行反引號 `` ` `` 裡的指令。
- 單引號比較「死板」，它會把裡面的所有字元都當成**普通的純文字**，不做任何額外處理。
	- **不會**解析變數，寫什麼就印什麼。

|**寫法**|**指令**|**輸出結果**|**說明**|
|---|---|---|---|
|**雙引號**|`echo "Hi $NAME"`|`Hi Gemini`|**會**找到變數並代入。|
|**單引號**|`echo 'Hi $NAME'`|`Hi $NAME`|**不會**理會變數，直接印出字面內容。|
## EX2
Assign
```Shell=
MY_OBFUSCATED_VARIABLE=Hello
```
and then
```Shell=
echo $MY_OSFUCATED_VARIABLE
```
Then you will ==get nothing== (as the second OBFUSCATED is mis-spelled).

## EX3
myvar2.sh code is
```Shell=
#!/bin/sh
echo "MYVAR is : $MYVAR"
MYVAR="hi there"
echo "MYVAR is : $MYVAR"
```
Next use the `export` command to make the variable available to child processes.
```Shell=
$ export MYVAR  
$ ./myvar2.sh  
MYVAR is: hello  
MYVAR is: hi there
```
## EX4
```Shell=
$ MYVAR=hello  
$ echo $MYVAR  
hello  
$ . ./myvar2.sh  
MYVAR is: hello  
MYVAR is: hi there  
$ echo $MYVAR  
hi there
```
## EX5
```Shell=
#!/bin/sh  
echo "What is your name?"  
read USER_NAME
echo "Hello $USER_NAME"  
echo "I will create you a file called ${USER_NAME}_file"  
touch "${USER_NAME}_file"
```
>Also note the quotes around `"${USER_NAME}_file"` - if the user entered "Steve Parker" (note the space) then without the quotes, the arguments passed to `touch` would be `Steve` and `Parker_file` - that is, we'd effectively be saying `touch Steve Parker_file`, which is two files to be `touch`ed, not one.

>在現代的 Linux 系統中，當你使用 `ls` 查看檔案時，如果檔名中包含 **空白鍵** 或 **特殊字元**，系統會自動在顯示時幫你加上單引號。
>這是為了讓你一眼看出：「這是一個完整的檔案，而不是兩個檔案」。
>解法 : 用 `ls -N`或 `printf "%s\n" *` *`

這就是為什麼在寫 Shell Script 時，老手通常會說：**「只要用到變數，除非有特殊理由，否則一律加上雙引號」**。除非你在處理密碼之類不要強制解析的字串。