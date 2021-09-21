
# PascalScript
Programming language strongly inspired from C++, with features from other languages that I like.

## Keywords

- [`switch`](#switch-case)
	- `case`
		- `case(default)`
		- `case(error)`
	- `likely`, `unlikely`
     - `continue`
- [`for`](#for-loops)
	- `in`
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
<details>
<summary> <h4>Detailed Syntax</h4> </summary>

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
		(memberof<rfe.begin()>(++operator) == true || memberof<rfe.begin()>(operator++))
		``` 

</details>
<br>

### Example
```c
for(int i = 0; i < 10; i++)
{
	std::println(i);
	
	if(i == 5)
		break;
};
```
```c
for(int i in [0 ... 9])
{
	std::println(i);

	if(for.index == 5)
		break;
};
```
```c
int i = 10;
for(i)
{
	std::println(10 - i--);
	
	if(for.index == 5)
		break;
};
``` 
```c
for(int[] array = [0 ... 9],
	int* ptr = array.begin();
	ptr < array.end(); ptr++)
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
	
### TODO: index attribute

### TODO: range-based loop


## Switch Case

### Syntax

<details>
<summary> <h4>Detailed Syntax</h4> </summary>

- switch_statement:
	> `switch`(*switchee*, <sup><sub>(optional)</sub></sup> *switch_epsilon* <sup><sub>(or)</sub></sup> *switch_hash_function*) *switch_block*

<br>

- switchee:
	- Expression resulting in a numerical value, an enumerated value, a floating-point / fixed-point value, hashable value or a runtime type (as a pointer type or reference type).

<br>

- switch_epsilon:
	- valid if `typeof(switch_epsilon) == typeof(switchee)`
	- Used only if *switchee* is a floating / fixed point-resulting expression
		```c
		std::is_floating_point(typeof(switchee)) == true ||
		std::is_fixed_point(typeof(switchee)) == true
		```

<br>

- switch_hash_function:
	- valid if *switch_hash_function* is a `constexpr` function taking *switchee* as a single parameter and returning an integer
		```c
		using shf = switch_hash_function,
		std::is_function(shf) == true &&
		std::is_constexpr(shf) &&
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
uint var = 0;
switch(var)
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
	- NaN
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

### TODO : Likely / Unlikely

<br>

### Floating / Fixed point support
Switch cases can switch on floating / fixed point numbers. An  ε must then be provided as 2nd argument to the switch keyword. (Otherwise, `typeof(switchee).epsilon` is used as default value).

```c
float var = 3.141592;
switch(var) 	// ε omited, float.epsilon used.
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
A proper epsilon value could be provided by switching the 2nd line for `switch(var, 0.0001)`. 
If the switch was executed with such an ε, the proper π case would be selected and executed.
If the 1st line was `float var = float.NaN;`, `float var = float.infinity;` or `float var = -float.infinity;`, the error case would be selected and executed.

<br>

### Hashing
Beside integers, enums and floating / fixed point values, it is also possible to switch on other hashable values, such a strings or arrays.
A hash function must then be provided as 2nd argument to the `switch` keyword. (Otherwise std::hash is used).
```c
std::string var = "Good morning";
switch(var /*, std::hash */)	// std::hash used by default
{
	case("Hello World")		// Conversion from const char* to std::string
		std::print("Message was Hello World");
	case(std::wstring("Good morning"))	// Conversion from std::wstring to std::string
		std::print("Message was good morning");
	case(1)				// Conversion from integer 1 to string "1"
		std::print("Message was 1");	
	case(null)			// Uses std::hash(null)
		std::print("No message");

	/* case(std::file{"dest.txt"}) */	// Compile-time error, impossible to convert a std::file hash into std::string hash at compile-time.
	/* case("Good morning") */		// Compile-time error, identical hash in two cases
	case(default)				// Covers the error case
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
		std::print("Land vehicle");
	case(instanceof(Boat))
		std::print("Sea vehicle");
};
```
Output:
> Land vehicle
