1. The `case` statement saves going through a whole set of `if .. then .. else` statements.
```shell 
#!/bin/sh

echo "Please talk to me ..."
while :
do
  read INPUT_STRING
  case $INPUT_STRING in
	hello)
		echo "Hello yourself!"
		;;
	bye)
		echo "See you again!"
		break
		;;
	*)
		echo "Sorry, I don't understand"
		;;
  esac
done
echo 
echo "That's all folks!"
```
- The options we understand are then listed and followed by a right bracket, as `hello)` and `bye)`.  This means that if `INPUT_STRING` matches `hello` then that section of code is executed, up to the double semicolon.
-  If we wanted to exit the script completely then we would use the command `exit` instead of `break`.
- The `*)`, is the default catch-all condition; it is not required, but is often **useful for debugging purposes** even if we think we know what values the test variable will have.
- The whole case statement is ended with `esac` (case backwards!) then we end the while loop with a `done`.