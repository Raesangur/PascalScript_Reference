
# Fonctionalités du PascalScript
Language de programmation fortement inspiré du C++, avec des fonctionalités d'autres languages que j'aime.

## Structures
Tous les segments de code (fonctions, conditions, classes, structures, initialisations ect.) sont dénotés par une `{` d'ouverture et une `};` de fermeture.

## Classes

## Variables

## Switch Case

### Syntaxe
Une switch case commence par le mot-clé `switch`, suivi par, entre `()`, la variable sur laquelle switcher, puis un bloc de code.
Par la suite, chaque cas est dénoté par `case(x)`, où x est la valeur connue à compile-time sur laquelle switcher. Chaque cas contient ensuite un bloc de code.

Le cas par défault est dénoté par `case(default)`, et un cas d'erreur de parsing est également couvert par `case(error)`. 

```c
uint var = 0;
switch(var)
{
	case(0) fallsthrough
	{
		std::println("Value was 0");
	};
	case(1)
	{
		std::println("Value below 2");
	};
	case(2 ... 9)
	{
		std::println("Range-case value");
	};
	case(default)
	{
		std::println("Default value");
	};
};
```
Cet exemple imprimerait:
> Value was 0
> Value below 2

### `case(error)`
Les cas d'erreurs sont spécifiques aux switch-cases qui emploient un hash ou qui travaillent avec des nombres à virgules.
À l'exemple précédent, si `case(error)` était ajouté, le programmeur recevrait un warning, car il ne peut pas y avoir d'erreurs de parsing pour un switch case.
Dans le cas où un cas d'erreur n'est pas fourni à un switch-case qui peut fail, le cas défaut sera utilisé en cas d'erreur.
Si ni un cas d'erreur ni un cas défaut ne sont fournis, le switch est bypassé.
À noter que des exceptions runtimes lancées à l'intérieur des différents cas ne trigger pas le cas d'erreur et nécessitent un vrai bloc `try`, le cas d'erreur ne couvre que les erreurs de parsing.

### Ranges
Permet d'utiliser des ranges de chiffres pour couvrir plusieurs cas.

### Fallthrough
Le mot-clé `fallsthrough` peut être utilisé pour supporter des cas qui passent au cas suivant après leur exécution.

### Decimal support
Les switch cases permettent de switcher sur des nombres à virgule flottantes et virgule fixe. Un ε doit alors être fourni comme 2e argument du mot-clé `switch`. Si le ε n'est pas fourni, `std::epsilon<type>` est pris comme valeur par défaut.

```c
float var = 3.141592;
switch(var) 	// ε omis, std::epsilon<float> utilisé.
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
Dans ce cas, on tomberait dans le cas défaut, car le epsilon est beaucoup trop petit.
On pourrait fournir un epsilon en changeant la 2e ligne par `switch(var, 0.0001)`. Si le switch était effectué avec un tel ε, on tomberait dans le cas de π.
Si la 1ère ligne était `float var = float.NaN;`, `float var = float.infinity;` ou `float var = -float.infinity;`, on tomberait dans le case(error)

### Hashs
Il est également possible de switcher sur d'autres types qui sont hashable, comme des strings ou tableaux.
On peut passer au switch case une fonction de switch personalisée avec le 2e argument du switch, case.
```c
std::string var = "Bon matin";
switch(var /*, std::hash */)	// std::hash<string> utilisée par défaut, pas besoin de l'indiquer
{
	case("Hello World")		// Conversion du const char* en std::string
	{
		std::print("Message was Hello World");
	};
	case(std::wstring("Bon matin"))	// Conversion du std::wstring en std::string
	{
		std::print("Message was Bon matin");
	};
	case(1)				// Conversion de l'entier 1 en string "1"
	{
		std::print("Message was 1");	
	};
	case(null)			// Utilise std::hash<string>(null)
	{
		std::print("No message");
	};
	/* case(std::file{"dest.txt"}){}; */	// Compile-time error, impossible de convertir un hash de std::file en std::string at compile-time.
	/* case("Bon matin"){}; */		// Compile-time error, hash identique présent à deux reprises
	case(default)				// Couvre le case(error)
	{
		std::print("No message or error parsing message");
	};
};
```
