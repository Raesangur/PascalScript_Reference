

# PascalScript
Programming language strongly inspired from C++, with features from other languages that I like.

## Keywords
- [`switch`](#switch-case)
	- `case`
		- `case(default)`
		- `case(error)`
	- `fallsthrough`
- types
	- `typeof`
	- `instanceof`
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


## Structures
Every code block is marked with an opening `{` and a closing `};`

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
	- Used only if *switchee* is a floating / fixed point-resulting expression (`std::is_floating_point(typeof(switchee)) == true || std::is_fixed_point(typeof(switchee)) == true`)

<br>

- switch_hash_function:
	- valid if *switch_hash_function* is a `constexpr` function taking *switchee* as a single parameter and returning an integer (`std::is_function(switch_hash_function) == true && std::is_constexpr(switch_hash_function) &&
std::parameters(switch_hash_function).count == 1 && std::parameters(switch_hash_function)[0].type == typeof(switchee) && std::is_integral(std::returns(switch_hash_function).type) == true`)'.
	
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
		- expression of type `typeof(switchee)` (if a hashing function is not used)
		- expression of type `std::returns(switch_hash_function).type` (otherwise)
	
<br>

- case_type_statement:
	>`case`(`instanceof`(*case_type_value*)) <sup><sub>(optional)</sub></sup> *case_attribute* *case_block*
		
<br>

- case_type_value:
	- valid if *switchee* is a pointer or reference of a type that inherits *case_type_value* (`typeof(switchee) inherits case_type_value == true && (std::is_pointer(typeof(switchee)) || std::is_reference(typeof(switchee)))`)

<br>

- case_attribute:
	- one of
		- `#fallsthrough`
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
	case(0) #fallsthrough
	{
		std::println("Value was 0");
	};
	case(1)
	{
		std::println("Value below 2");
	};
	case(2 ... 9) #likely
	{
		std::println("Range-case value");
	};
	case(default)
	{
		std::println("Default value");
	};
};
```
Output:
> Value was 0 <br>
> Value below 2

<br>

### `case(error)`
Les cas d'erreurs sont spécifiques aux switch-cases qui emploient un hash ou qui travaillent avec des nombres à virgules.
Error cases work with switch cases switching on an enumerated value, floating / fixed point values, or with hashes.
In the previous example, if `case(error` was added, a warning would be produced, because there cannot be errors with the parsing of integer-based switch cases.
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
The attribute `fallsthrough`can be used to support cases that jump to the following cases after their execution.

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
	{
		std::println("irrational √2");
	};
	case(std::phi)
	{
		std::println("irrational golden number");
	};
	case(std::e)
	{
		std::println("irrational e");
	};
	case(std::pi)
	{
		std::println("irrational π");
	};
	case(default) 
	{
		std::println("Not in supported irrational list");
	};
	case(error)
	{
		std::println("Error while parsing number");
	};
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
	{
		std::print("Message was Hello World");
	};
	case(std::wstring("Good morning"))	// Conversion from std::wstring to std::string
	{
		std::print("Message was good morning");
	};
	case(1)				// Conversion from integer 1 to string "1"
	{
		std::print("Message was 1");	
	};
	case(null)			// Uses std::hash(null)
	{
		std::print("No message");
	};
	/* case(std::file{"dest.txt"}){}; */	// Compile-time error, impossible to convert a std::file hash into std::string hash at compile-time.
	/* case("Bon matin"){}; */		// Compile-time error, identical hash in two cases
	case(default)				// Covers the error case
	{
		std::print("No message or error parsing message");
	};
};

<br>

```
### Runtime type checking
Besides values, switch statements can also be used to switch between types at runtime, specifically polymorphic instances through pointer or reference types.
By using a type as switchee expression in the `switch` statement, it is then possible to use `case(instanceof(someOtherType))` cases. In the following example, class `Vehicle` is defined and classes `Truck`, `Bicycle` and `Boat` are created to inherit from class `Vehicle`.
```c
Vehicle* myTruck = new Truck();
switch(myTruck)
{
	case(instanceof(Bicycle)) #fallsthrough
	case(instanceof(Truck))
	{
		std::print("Land vehicle");
	};
	case(instanceof(Boat))
	{
		std::print("Sea vehicle");
	};
};
```
Output:
> Land vehicle
