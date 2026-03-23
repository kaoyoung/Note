### Reference [Mastering Casting Operators in C++: A Comprehensive Guide](https://medium.com/@youssef.sabr/mastering-casting-operators-in-c-a-comprehensive-guide-34e219b0c038)
# 1. Implicit Casting
Implicit casting, also called **type coercion**, is a type of casting that is **automatically performed by the C++ compiler** when it can safely convert one data type into another without losing information or precision.
- It is **performed by the compiler** to ensure that expressions involving different data types can be evaluated correctly.
- Implicit casting typically occurs **when you mix different data types in expressions or assignments**.
- It is **safe** and **does not lead to data loss or precision issues**.

For example 
```c++
#include <iostream>

int main(){
    int num1 = 5;
    double num2 = 2.5;
    double result = num1 + num2;

    std::cout << num1 << " " << num2 << " " << result << std::endl;
    return 0;
}             
```
The output is 
```text
5 2.5 7.5
```
# 2. Explicit Casting
### C++ provides 4 type of explicit casting operators
- static\_cast : Used for most general type conversions with compile-time checking.
- dynamic\_cast :  Primarily used for **handling polymorphism and working with class hierarchies**. It performs **runtime type checking** and is often used with pointers or references to base and derived classes.
- const\_cast :  Used for low-level type conversions, typically involving pointers or integers. It can be dangerous if used incorrectly and **should be used with caution**.
- reinterpret\_cast :  Used to add or remove the `const` qualifier from a variable.
Explicit casting, also known as **type casting**, is a type of casting where the **programmer explicitly specifies** the type conversion using casting operators.

## **Static Cast (**`static_cast`**):**
 It is a versatile operator that can handle many different types of conversions during compile-time. Here's a comprehensive explanation of how `static_cast` works:
### Syntax
 ```c++
new_type static_cast <new_type> (expression);
```
- `new_type`: The target data type to which you want to convert the expression.
- `<new_type>`: An optional type name enclosed in angle brackets (<>). It is not needed in most cases, as the compiler can infer the target type.
- `expression`: The value or expression that you want to convert.
### Purpose
The primary purpose of `static_cast` is to perform type conversions in a controlled and explicit manner.
### Example
1. Numeric Type Conversion
```c++
double num = 3.141592;  
int integerPart = static_cast<int>(num);
```
2. Converting Enum to Underlying Type
```c++
enum class Color { Red, Green, Blue };  
int colorValue = static_cast<int>(Color::Green);
```
- we convert the enum value `Color::Green` to its underlying integer value, which is `1` in this case.
3. User-Defined Type Conversion
```c++
class MyType {  
public:  
	operator int() const {  
	return 42;  
	}  
}

int main(){
	MyType myObject;  
	int intValue = static_cast<int>(myObject);
}
```

### Note
1. **Safety and Compile-Time Checks** : It performs compile-time type checking. This means that if the conversion is not safe or possible, the compiler will generate an error, helping to catch type-related issues early during development.
2. **Its limitations** :  It cannot be used for conversions involving classes with inheritance hierarchies or for certain types of pointer conversions (e.g., casting between unrelated class pointers).
	- If you have a Virtual Base Class, the memory layout is determined at **runtime**, not compile time. Therefore, `static_cast` (which is compile-time only) cannot handle the conversion down from a virtual base to a derived class.

## Dynamic Cast (`dynamic_cast`):
It allows you to perform safe **runtime** type checking and type conversions when dealing with pointers or references to **base and derived classes**.
### Syntax
```c++
dynamic_cast<new_type>(expression)
```
- `new_type`: The target type to which you want to cast the expression.
- `expression`: The expression or pointer/reference that you want to cast.
### Key feature
- `dynamic_cast` is commonly used in situations where you have a **base class pointer or reference that may point to a derived class object within a class hierarchy**.
- It helps determine whether the type conversion is valid at **runtime**.
### Safety
- `dynamic_cast` performs a runtime type check to ensure that the conversion is safe.
- If the conversion is not valid, it returns a null pointer (for pointer casts) or throws a `std::bad_cast` exception (for reference casts).
### Polymorphism:
- `dynamic_cast` is particularly useful when working with polymorphic types (i.e., classes with virtual functions).
- It allows you to safely cast from a base class pointer or reference to a derived class pointer or reference.
### Example
```c++
class Base {  
public:  
	virtual void print() {  
		std::cout << "Base" << std::endl;  
	}  
};  
class Derived : public Base {  
public:  
	void print() override {  
		std::cout << "Derived" << std::endl;  
	}  
};

int main(){
	Base* basePtr = new Derived;  
  
	// Safe dynamic_cast to Derived*  
	Derived* derivedPtr = dynamic_cast<Derived*>(basePtr);  
  
	if (derivedPtr) {  
	// Conversion successful  
	derivedPtr->print(); // Calls Derived::print()  
	} else {  
	// Conversion failed  
	}
}
```
### Considerations
- `dynamic_cast` can only be used with **pointers and references**.
- It is applicable to polymorphic types with **at least one** virtual function in the base class.
- For `dynamic_cast` to work correctly, the base class must have a polymorphic type, meaning it should have **at least one** virtual function.
- It is **slower** than other casting operators like `static_cast` because it involves **runtime type checking**.

## Reinterpret Cast (`reinterpret_cast`):
`reinterpret_cast` **merely reinterprets the bits in memory**. It does not change the data itself, nor does it adjust pointer offsets; it simply takes the memory address originally pointing to Object A and forcibly slaps a 'This is Object B' label on it.
### Syntax
```c++
reinterpret_cast<new_type>(expression)
```
- `new_type`: The target type to which you want to cast the expression.
- `expression`: The expression or pointer that you want to cast.
### Key feature
1.  Low-Level Type Conversion:
-  `reinterpret_cast` is used when you need to perform type conversions that are not directly supported by other casting operators, such as converting between unrelated pointer types or changing the type of an integer.
2. Pointer Type Conversions:
- This operator can be used to reinterpret the memory layout of an object, which can be dangerous if not done carefully.
3. Integer to Pointer Conversion:
- You can use `reinterpret_cast` to convert an integer value into a pointer and vice versa.
### Example 
1. Pointer Conversion:
```c++
int num = 42;  
double* ptr = reinterpret_cast<double*>(&num);
```
2. Integer to Pointer Conversion:
```c++
uintptr_t address = reinterpret_cast<uintptr_t>(&num);  
int* numPtr = reinterpret_cast<int*>(address);
```

## Const Cast (`const_cast`):
### Syntax
```c++
const_cast<new_type>(expression)
```
- `new_type`: The target type to which you want to cast the expression.
- `expression`: The expression or pointer/reference that you want to cast.
### Example 
1.  **Removing** `const`:
```c++
const double pi = 3.141592;  
double* nonConstPtr = const_cast<double*>(&pi); // Remove const-ness  
*nonConstPtr = 4.0; // Modifying the non-const version is undefined behavior
```
### Considerations
1. Modifying a const object through a non-const pointer obtained via `const_cast` results in undefined behavior, so ensure that you only use `const_cast` when you genuinely need to modify the non-const version of an object.

## Conclusion
|**Cast Type**|**Values (Data)**|**Pointers (Addresses)**|**References (Aliases)**|**Notes**|
|---|---|---|---|---|
|**`static_cast`**|✅ **Common** (`int`↔`float`)|✅|✅|Most versatile; compile-time checks.|
|**`const_cast`**|❌|✅|✅|Exclusively for removing `const`/`volatile`.|
|**`dynamic_cast`**|❌|✅|✅|For Polymorphism. **Throws Exception** on reference failure.|
|**`reinterpret_cast`**|⚠️ (Int↔Ptr only)|✅|✅|Reinterprets memory bits blindly.|
Only `static_cast` truly handles "data conversion" (math/values). The other casts are mostly concerned with "how we view this memory" (identity conversion), which is why they are primarily used with pointers and references.