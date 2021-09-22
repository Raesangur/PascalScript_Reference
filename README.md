
# PascalScript
Programming language strongly inspired from C++, with features from other languages that I like.

## Keywords

- [`switch`](#switch-case)
    - `case`
        - `case(default)`
        - [`case(error)`](#case(error))
    - [`likely`, `unlikely`](#likely-unlikely)
     - [`continue`](#fallthrough)
- [`for`](#for-loops)
    - [`in`](#range-based-loop)
- metadata & reflections
    - `typeof`
    - `sizeof`
    - `alignof`
    - `nameof`
    - `accessof`
    - `scopeof`
    - `instanceof`
    - `memberof<>`
- types
    - casts
        - `cast<>`
        - `force_cast<>`
        - `dyn_cast<>`
    - `inherits`
    - fundamental types
        - `uint`, `int`
        - `uint8`, `uint16`, `uint32`, `uint64`, `int8`, `int16`, `int32`, `int64`
        - `ulong`, `long`
        - `ufloat`, `float`
        - `ufloat16`, `ufloat32`, `ufloat64`, `float16`, `float32`, `float64`
        - `bool`
        - `void`
        - `char`, `char8`, `char16`, `char32`
        - `byte`, `bit`
- misc
    - `using`


## Structures
Every code block is marked with an opening `{` and a closing `};`

## For loops
### Syntax
<details open>
    <summary> Detailed Syntax </summary>

- for_statement:
    > `for`(<sup><sub>(optional)</sub></sup> *for_init_statement*; *for_condition*; <sup><sub>(optional)</sub></sup> *for_iteration_expression*) *for_block*
    - or 
    > `for`(<sup><sub>(optional)</sub></sup> *for_init_statement*; *ranged_for_declaration* `in` *ranged_for_expression* ) *for_block*

<br>

- for_init_statement:
    - <sup><sub>(1 ... n)</sub></sup> comma-separated declarations.
    - If used, must end in a `;`. If unused, `;` may be skipped.
    - Will be executed once before the beginning of the loop.
    - Variables declared in the *for_init_statement* in the same scope as variables declared in the *for_block*.

<br>

- for_condition:
    - A boolean-resulting expression.
    - This expression is evaluated at the beginning of every iteration of the loop, including the first one.

<br>

- for_iteration_expression:
    - <sup><sub>(1 ... n)</sub></sup> comma-separated expressions to be executed at the end of the *for_block*.
    - If unused, the preceding `;` may be skipped.
    
<br>

- for_block:
    > {
    > &nbsp;&nbsp;&nbsp;&nbsp;<sup><sub>(0 ... n)</sub></sup> statement
    > };
    
<br>

- ranged_for_declaration:
    - A declaration of a variable of type `typeof(*ranged_for_expression.begin())` or `typeof(*ranged_for_expression.begin())&`.
    - The variable declared in the *ranged_for_declaration* will be updated before each iteration of the loop
    
<br>

- ranged_for_expression:
    - An expression which resulting value has a `begin()` and `end()` function defined, that both return an object defining the `++operator` or the `operator++`
        ```c
        using rfe = ranged_for_expression,
        memberof<rfe>(begin()) == true &&
        memberof<rfe>(end()) == true &&
        std::returns(rfe.begin()).type == std::returns(rfe.end()).type &&
        (memberof<rfe.begin()>(++operator()) == true || memberof<rfe.begin()>(operator++())) &&
        memberof<rfe.begin()>(operator==(typeof(rfe.begin())) == true
        ``` 

</details>
<br>

### Example
```c
for(int i = 0; i <= 5; i++) // Regular C-style for loop
    std::println(i);
```
```c
for(int i in [0 ... 9])     // Ranged-based for loop
{
    std::println(i);

    if(for.index == 5)
        break;
};
```
```c
int i = 6;
for(i)              // For loop with only condition filled in
    std::println(6 - i--);
``` 
```c
for(int[] array = [0 ... 9],    // Two initialized values & two iteration expressions
    int* ptr = array.begin();
    ptr < array.end(); *ptr = 0, ptr++)
{
    std::println(*ptr);

    if(for.index == 5)
        break;
};
```

All output:
> 0 <br>
> 1 <br>
> 2 <br>
> 3 <br>
> 4 <br>
> 5
    
<br>

### range-based loop
`for` statements can also be used to loop over an iterating range of values, such as an array, a string literal or any standard library containers.
The keyword `in` is used to denote the *ranged_for_declaration* from the *ranged_for_expression*.
These ranged-based `for` loops need the iterating object to have the `begin()` and `end()` methods defined, that both return an object able to iterate with the `++operator` or `operator++` and compare with the `operator==`.
These range-based `for` loops will simply expand in a regular `for` loop, where syntax (1) is equivalent to syntax (2).
```c
// Syntax (1)
for(int i in [0 ... 9])
{
	std::println(i);
};

// Syntax (2)
// ranged_for_expression was of type int[]&&;
// int[].begin() and int[].end() returns an int*
for(int[]&& __expression = [0 ... 9],
    int* __it = __expression.begin();
    __it < __expression.end();
    ++__it)    // If ++operator() is not available for iterator type, will try with operator++()
{
    int i = *it;
    // ranged-based for loop statements begin here:
    std::println(i);
};
```

<br>

### `for.index` variable
The `for.index` variable is an easy way to keep track of the loop index during ranged-based `for` loops (or any non-index-based `for` loops). 
This variable is normally uninitialized, but if used, will keep track of the current loop index.
For range-based `for` loops, the `for.index` variable will be calculated from the begin position and the loop iterator (see example (1)).
For other loops, there are two possibilities:
    - If the `for` loop contains a simple `for(someType i = someValue; i < someOtherValue; i++)`, and `memberof<someType>(operator-(typeof(someValue))) == true`, the `for.index` variable will be calculated from the used loop index `i` as long as this index is not modified during the loop execution (see example(2)).
    - If the `for` loop is more complicated, has an index that can't be easily calculated from or modifies the index during execution, the `for.index` variable will be calculated independently (see example(3)).

```c
// EXAMPLE (1)
// Syntax (1)
std::println("EXAMPLE 1 - Syntax 1");
for(int i in [0 ... 5])
{
	std::print(for.index + i);
};
// Syntax (2) <- Will be generated from Syntax (1)
std::println("EXAMPLE 1 - Syntax 2");
for(int[]&& __expression = [0 ... 5],
	int* __begin = __expression.begin(),
    int* __it = __begin;
    __it < __expression.end(); ++__it)
{
    int i = *it;
    std::print((__it - __begin) + i);	    // for.index is simply substituted
};

// EXAMPLE (2)
// Syntax (1)
std::println("\nEXAMPLE 2 - Syntax 1");
for(int i = 5; i < 10; i++)
    std::print(i - for.index);

// Syntax (2) <- Will be generated from Syntax (1)
std::println("EXAMPLE 2 - Syntax 2");
for(int __begin = 5, int i = 5; i < 10; i++)
    std::print(i - (i - __begin));

// EXAMPLE (3)
// Syntax (1)
std::println("\nEXAMPLE 3 - Syntax 1");
for(int i = 0; i < 5; i++)
{
	std::print(i + for.index);
	
	if (for.index == 3)
		i = 5;
};

// Syntax (2) <- Will be generated from Syntax (1)
// for loop indexing variable and modified during execution, a standalone index will be generated
std::println("EXAMPLE 3 - Syntax 2");
for(int __index = 0, int i = 0; i < 5; __index++, i++)
{
	std::print(i + __index);

	if(__index == 3)
		i = 5;
};
```
Output:
>	EXAMPLE 1 - Syntax 1
>  0246810
>  EXAMPLE 1 - Syntax 2
>  0246810
>  
>  EXAMPLE 2 - Syntax 1
>  55555
>  EXAMPLE 2 - Syntax 2
>  55555
>  
>  EXAMPLE 3 - Syntax 1 
>  0246
>  EXAMPLE 3 - Syntax 2
>  0246

<br>

## Switch Case

### Syntax

<details open>
    <summary> Detailed Syntax </summary>

- switch_statement:
    > `switch`(*switchee*; <sup><sub>(optional)</sub></sup> *switch_epsilon* <sup><sub>(or)</sub></sup> *switch_hash_function*) *switch_block*

<br>

- switchee:
    - Expression resulting in a numerical value, an enumerated value, a floating-point / fixed-point value, hashable value or a runtime type (as a pointer type or reference type).

<br>

- switch_epsilon:
    - valid if `typeof(switch_epsilon) == typeof(switchee)`
    - If unused, the preceding `;` may be skipped.
    - Used only if *switchee* is a floating / fixed point-resulting expression
        ```c
        std::is_floating_point(typeof(switchee)) == true ||
        std::is_fixed_point(typeof(switchee)) == true
        ```

<br>

- switch_hash_function:
    - If unused, the preceding `;` may be skipped.
    - valid if *switch_hash_function* is a `constexpr` function taking *switchee* as a single parameter and returning an integer
        ```c
        using shf = switch_hash_function,
        std::is_function(shf) == true &&
        std::is_constexpr(shf) == true &&
        std::parameters(shf).count == 1 &&
        std::parameters(shf)[0].type == typeof(switchee) &&
        std::is_integral(std::returns(shf).type) == true
        ```
    
<br>

- switch_block:
    > {
    > &nbsp;&nbsp;&nbsp;&nbsp; <sup><sub>(1 ... n)</sub></sup> *case_statement* <sup><sub>(or)</sub></sup> *case_type_statement*
    > };

<br>

- case_statement:
    > `case`(*case_value*) <sup><sub>(optional)</sub></sup> *case_attribute* *case_block* 
    
<br>

- case_value:
    - one of
        - `default`
        - `error`
        - expression of type `typeof(switchee)`. 
    
<br>

- case_type_statement:
    >`case`(`instanceof`(*case_type_value*)) <sup><sub>(optional)</sub></sup> *case_attribute* *case_block*
        
<br>

- case_type_value:
    - valid if *switchee* is a pointer or reference of a type that inherits *case_type_value* 
        ```c
        typeof(switchee) inherits case_type_value == true &&
        (std::is_pointer(typeof(switchee)) || std::is_reference(typeof(switchee)))
        ```

<br>

- case_attribute:
    - one of
        - `#likely`
        - `#unlikely`

<br>

- case_block:
    > {
    > &nbsp;&nbsp;&nbsp;&nbsp;<sup><sub>(0 ... n)</sub></sup> statement
    > };
</details>

<br>

#### Example:
```c
switch(uint var = 0)
{
    case(0)
    {
        std::println("Value was 0");
        continue;
    };
    case(1)
        std::println("Value below 2");
    case(2 ... 9) #likely
        std::println("Range-case value");
    case(default)
        std::println("Default value");
};
```
Output:
> Value was 0 <br>
> Value below 2

<br>

### `case(error)`
Error cases work with switch cases switching on an enumerated value, floating / fixed point values, or with hashes.
In the previous example, if `case(error)` was added, a warning would be produced, because there cannot be errors with the parsing of integer-based switch cases.
If no error cases are provided, the default case would be used in case of an error.
If neither error of default cases are provided, the switch case would be bypassed in case of an error.
Runtime exceptions thrown from different cases do not trigger the error cases. These cases are strictly for parsing errors.
The error case is hit in the following conditions:
- for a floating / fixed point value:
    - any NaN value
    - infinity
    -  -infinity
- for an enum:
    - value not in enum range
- for a hashable
    - exception thrown during hashing

<br>

### Ranges
With integer-based switch cases, allow for ranges of cases to be used with the `...` operator.

<br>

### Fallthrough
Cases do not fall through by default. To jump from another case to the next, the `continue` keyword can be used to jump to the next case in order. 
This continue can be located within a subbranch of the case. 

<br>

### Likely Unlikely
The `#likely` and `#unlikely` attributes allow for smart compiler optimizations, telling the compiler which switch case branch is the most or least likely to occur, so that the compiler can place it accordingly in the execution chain. Optimization behavior is not guaranteed.

<br>

### Floating / Fixed point support
Switch cases can switch on floating / fixed point numbers. An  ε must then be provided as 2nd argument to the switch keyword. (Otherwise, `typeof(switchee).epsilon` is used as default value).

```c
float var = 3.141592;
switch(var)     // ε omited, float.epsilon used.
{
    case(std::sqrt(2))
        std::println("irrational √2");
    case(std::phi)
        std::println("irrational golden number");
    case(std::e)
        std::println("irrational e");
    case(std::pi)
        std::println("irrational π");
    case(default) 
        std::println("Not in supported irrational list");
    case(error)
        std::println("Error while parsing number");
};
```
In this example, the default case would be used, since the epsilon value is incredibly small. 
A proper epsilon value could be provided by switching the 2nd line for `switch(var; 0.0001)`. 
If the switch was executed with such an ε, the proper π case would be selected and executed.
If the 1st line was `float var = float.NaN;`, `float var = float.infinity;` or `float var = -float.infinity;`, the error case would be selected and executed.

<br>

### Hashing
Beside integers, enums and floating / fixed point values, it is also possible to switch on other hashable values, such a strings or arrays.
A hash function must then be provided as 2nd argument to the `switch` keyword. (Otherwise std::hash is used).
```c
std::string var = "Good morning";
switch(var /*; std::hash */)    // std::hash used by default
{
    case("Hello World")     // Conversion from const char* to std::string
        std::print("Message was Hello World");
    case(std::wstring("Good morning"))  // Conversion from std::wstring to std::string
        std::print("Message was good morning");
    case(1)             // Conversion from integer 1 to string "1"
        std::print("Message was 1");    
    case(null)          // Uses std::hash(null)
        std::print("No message");

    /* case(std::file{"dest.txt"}) */   // Compile-time error, impossible to convert a std::file hash into std::string hash at compile-time.
    /* case("Good morning") */      // Compile-time error, identical hash in two cases
    case(default)           // Covers the error case
        std::print("No message or error parsing message");
};
```

<br>

### Runtime type checking
Besides values, switch statements can also be used to switch between types at runtime, specifically polymorphic instances through pointer or reference types.
In the following example, class `Vehicle` is defined and classes `Truck`, `Bicycle` and `Boat` are created to inherit from class `Vehicle`.
Type checking cases and value checking cases cannot be mixed in the same switch statement. 
```c
Vehicle* myTruck = new Truck();
switch(myTruck)
{
    case(instanceof(Bicycle))
        continue;
    case(instanceof(Truck))
        std::println("Land vehicle");
    case(instanceof(Boat))
        std::println("Sea vehicle");
};
```
Output:
> Land vehicle
