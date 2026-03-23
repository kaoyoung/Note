1. There are a set of variables which are set for you already, and most of these cannot have values assigned to them.
2. The first set of variables we will look at are `$0 .. $9` and `$#`.
- The variable `$0` is the _basename_ of the program as it was called.
- `$1 .. $9` are the first 9 additional parameters the script was called with.
- `$#` is the number of parameters the script was called with.
```shell
#!/bin/sh
echo "I was called with $# parameters"
echo "My name is $0"
echo "My first parameter is $1"
echo "My second parameter is $2"
echo "All parameters are $@"
```
Let's look at running this code and see the output:
```text
$ /home/steve/var3.sh
I was called with 0 parameters
My name is /home/steve/var3.sh
My first parameter is
My second parameter is    
All parameters are 
$
$ ./var3.sh hello world earth
I was called with 3 parameters
My name is ./var3.sh
My first parameter is hello
My second parameter is world
All parameters are hello world earth
```

3. We can take more than 9 parameters by using the `shift` command; look at the script below:
```shell
#!/bin/sh
while [ "$#" -gt "0" ]
do
  echo "\$1 is $1"
  shift
done
```
This script keeps on using `shift` until `$#` is down to zero, at which point the list is empty.

4. Another special variable is `$?`. This contains the exit value of the last run command. So the code:
```shell
#!/bin/sh
/usr/local/bin/my-command
if [ "$?" -ne "0" ]; then
  echo "Sorry, we had a problem there!"
fi
```
will attempt to run `/usr/local/bin/my-command` which should exit with a value of zero if all went well, or a nonzero value on failure. We can then handle this by checking the value of `$?` after calling the command.

5. The `$$` variable is the PID (Process IDentifier) of the currently running shell.
- This can be useful for creating temporary files, such as `/tmp/my-script.$$` which is useful if many instances of the script could be run at the same time, and they all need their own temporary files.

6. The `$!` variable is the PID of the last run background process.
- This is useful to keep track of the process as it gets on with its job.

7. Another interesting variable is `IFS`. This is the _Internal Field Separator_. The default value is `SPACE TAB NEWLINE`, but if you are changing it, it's easier to take a copy, as shown:
```shell
#!/bin/sh
old_IFS="$IFS"
IFS=:
echo "Please input some data separated by colons ..."
read x y z
IFS=$old_IFS
echo "x is $x y is $y z is $z"
```
This script runs like this
```
$ ./ifs.sh
Please input some data separated by colons ...
hello:how are you:today
x is hello y is how are you z is today
```
More example
```text
$ ./ifs.sh
Please input some data separated by colons ...
hello:how are you:today:my:friend
x is hello y is how are you z is today:my:friend
```
- 為啥不寫`old_IFS=$IFS`
	- 工程師養成的習慣就是：**「只要變數是字串，一律加雙引號」**，這樣永遠不會錯。
	- It is important when dealing with IFS in particular (but any variable not entirely under your control) to realise that it could contain spaces, newlines and other "uncontrollable" characters. It is therefore a very good idea to use double-quotes around it, ie: `old_IFS="$IFS"` instead of `old_IFS=$IFS`.
		- \[Specific Case\], but \[General Rule\]
			- It’s dangerous to drive fast in the rain (but any bad weather, really).
