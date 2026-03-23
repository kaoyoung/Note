1. `for` loops iterate through a set of values until the list is exhausted:
What will be the output of this code
```shell
#!/bin/sh
for i in hello 1 * 2 goodbye 
do
  echo "Looping ... i is set to $i"
done
```
The output is
```text
Looping ... i is set to hello
Looping ... i is set to 1
Looping ... i is set to (name of first file in current directory)
    ... etc ...
Looping ... i is set to (name of last file in current directory)
Looping ... i is set to 2
Looping ... i is set to goodbye
```
How about this code 
```shell
#!/bin/sh
for i in hello 1 \* 2 goodbye 
do
  echo "Looping ... i is set to $i"
done
```
The output is 
```text
Looping ... i is set to hello
Looping ... i is set to 1
Looping ... i is set to *
Looping ... i is set to 2
Looping ... i is set to goodbye
```

2. `while` loop is similar to the while loop in c
```shell
#!/bin/sh
INPUT_STRING=hello
while [ "$INPUT_STRING" != "bye" ]
do
  echo "Please type something in (bye to quit)"
  read INPUT_STRING
  echo "You typed: $INPUT_STRING"
done
```

>What happens here, is that the echo and read statements will run indefinitely until you type "bye" when prompted.

3. In `while` loop, the colon (`:`) always evaluates to true. It is ofter preferable to use a real exit condition.
```shell
#!/bin/sh
while :
do
  echo "Please type something in (^C to quit)"
  read INPUT_STRING
  echo "You typed: $INPUT_STRING"
done
```

4. One useful trick is the `while read` loop.
```shell
#!/bin/sh
cat myfile.txt | while read input_text
do
    case ${input_text} in
        hello)          echo English ;;
        howdy)          echo American ;;
        gday)           echo Ausralian ;;
        bonjour)        echo French ;;
        "guten tag")    echo German ;;
        *)              echo Unknown Language: ${input_text} ;;
    esac
done
```
This reads the file "`myfile.txt`", one line at a time, into the variable "`$input_text`". The case statement then checks the value of `$input_text`.
- Each line must end with a LF (newline) - if `cat myfile.txt` doesn't end with a blank line, that final line will not be processed.
```shell
#!/bin/sh
cat myfile.txt | while read input_text || [ -n "$input_text" ]
do
    case ${input_text} in
        hello)          echo English ;;
        howdy)          echo American ;;
        gday)           echo Ausralian ;;
        bonjour)        echo French ;;
        "guten tag")    echo German ;;
        *)              echo Unknown Language: ${input_text} ;;
    esac
done
```
This updated code correctly processes the final line even if it lacks a trailing newline.



