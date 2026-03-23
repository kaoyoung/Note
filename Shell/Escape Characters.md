1. The use of double quotes (`"`) characters affect how spaces and TAB characters are treated
```bash 
$ echo Hello       World 
Hello World 
$ echo "Hello       World" 
Hello       World 
```
What will be the output of the following code 
```bash 
$ echo "Hello       "World"" 
```
This code would be interpreted as three parameters:
- "Hello       "
- World
- ""
So the output is 
```bash 
Hello       World
```
2. Most characters (`*`, `'`, etc) are not interpreted (ie, they are taken literally) by means of placing them in double quotes ("")
- asterisk (`*`)
```bash 
$ echo *
Desktop Documents Downloads espeon2eevee.fifo Music Pictures Public Templates test_monarch.py Videos
$ echo "*"
*
```
- In the first example, `*` is expanded to mean all files in the current directory.
- In the second, we put the `*` in double quotes, and it is interpreted literally.
3. However, `"`, `$`, `` ` ``, and `\` are still interpreted by the shell, even when they're in double quotes.
```bash 
$ echo "A quote is \", backslash is \\, backtick is \`."
A quote is ", backslash is \, backtick is `.
$ echo "A few spaces are    and dollar is \$. \$X is ${X}."
A few spaces are    and dollar is $. $X is 5.
```
- Backslash (`\`) character is used to mark these special characters so that they are not interpreted by the shell
- Dollar (`$`) is special because it marks a variable, so `$X` is replaced by the shell with the contents of the variable `X`.