 1. Consider the following code snippet which prompts the user for input, but accepts defaults:
 ```shell
 #!/bin/sh
echo -en "What is your name [ `whoami` ] "
read myname
if [ -z "$myname" ]; then
  myname=`whoami`
fi
echo "Your name is : $myname"
 ```
 - `-z` : Zero length
 - `` `whoami` ``  : Commands inside backticks (like `` `whoami` ``) run in a "subshell." This means if you run a command like `cd` inside those backticks, it won't change the directory of your main script.
 
 2. By using curly braces and the special ":-" usage, you can specify a default value to use if the variable is unset:
 ```Shell
echo -en "What is your name [ `whoami` ] "
read myname
echo "Your name is : ${myname:-`whoami`}"
 ```

3.  There is another syntax, ":=", which sets the variable to the default if it is undefined:
```shell
echo "Your name is : ${myname:=John Doe}"
```

|**Syntax**|**Action**|**Does it change the variable?**|
|---|---|---|
|**`${var:-word}`**|Use `word` if `var` is empty.|**No**|
|**`${var:=word}`**|Set `var` to `word` if it was empty.|**Yes**|
