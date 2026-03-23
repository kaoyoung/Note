1. `test` is not often called directly. `test` is more frequently called as `[`. `[` is a symbolic link to `test`, just to make shell programs more readable.
```shell
$ type [
[ is a shell builtin
$ which [
/usr/bin/[
$ ls -l /usr/bin/[
lrwxrwxrwx 1 root root 4 Mar 27 2000 /usr/bin/[ -> test
$ ls -l /usr/bin/test
-rwxr-xr-x 1 root root 35368 Mar 27  2000 /usr/bin/test
```

2. This means that '`[`' is actually a program, just like `ls` and other programs, so it must be surrounded by spaces:
This code ==won't work==
```shell
if [$foo = "bar" ]
```
- it is interpreted as `if test$foo = "bar" ]`, which is a '`]`' without a beginning '`[`'.

3. The syntax for `if...then...else...` is:
```text
if [ ... ]
then
  # if-code
else
  # else-code
fi
```
Note that `fi` is `if` backwards! This is used again later with `case` and `esac`.

4. The "`if [ ... ]`" and the "`then`" commands must be ==on different lines==.
```text
if [ ... ]
then
  # if-code
fi
```

5. Alternatively, the semicolon "`;`" can separate them:
```text
if [ ... ]; then
  # do something
fi
```

6. You can also use the `elif`, like this:
```text
if  [ something ]; then
echo "Something"
elif [ something_else ]; then
  echo "Something else"
else
   echo "None of the above"
fi
```
This will `echo "Something"` if the `[ something ]` test succeeds, otherwise it will test `[ something_else ]`, and `echo "Something else"` if that succeeds. If all else fails, it will `echo "None of the above"`.
**Note that `fi` should be written in the last.**

7. The backslash (`\`) is used to split the single-line command across two lines in the shell script file, for readability purposes
The following two codes are equivalent
```shell
if [ "$X" -nt "/etc/passwd" ]; then
      echo "X is a file which is newer than /etc/passwd"
fi
```

```shell
[ "$X" -nt "/etc/passwd" ] && \
      echo "X is a file which is newer than /etc/passwd"
```

8. If you want your shell script to behave more gracefully, you will have to check the contents of the variable before you test it - maybe something like this:
```shell
echo -en "Please guess the magic number: "
read X
echo $X | grep "[^0-9]" > /dev/null 2>&1
if [ "$?" -eq "0" ]; then
  # If the grep found something other than 0-9
  # then it's not an integer.
  echo "Sorry, wanted a number"
else
  # The grep found only 0-9, so it's an integer. # We can safely do a test on it.
  if [ "$X" -eq "7" ]; then
    echo "You entered the magic number!"
  fi
fi
```
- echo 
	- -n : no newline
	- -e : 启用转义字符解释
	- -E : 禁用转义字符解释（默认）
	- echo "hello\n"的輸出是 : hello\n
- `grep [0-9]` finds lines of text which contain digits (0-9) and possibly other characters, so the caret (`^`) in `grep [^0-9]` finds only those lines which don't consist only of numbers.

9. We can use test in while loops as follows:
```shell
#!/bin/sh
X=0
while [ -n "$X" ]
do
  echo "Enter some text (RETURN to quit)"
  read X
  echo "You said: $X"
done
```
- -n : **`-n`**：這是 **"Non-zero"** (非零) 的縮寫。



