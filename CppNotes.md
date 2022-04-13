# C++ Notes


### N1. enum class vs enum

#### 1. They don't convert implicitly to int.
```cpp
    enum Color { red, green, blue };                    // plain enum 
    enum Card { red_card, green_card, yellow_card };    // another plain enum 
    enum class Animal { dog, deer, cat, bird, human };  // enum class
    enum class Mammal { kangaroo, deer, human };        // another enum class

    // examples of bad use of plain enums:
    Color color = Color::red;
    Card card = Card::green_card;

    int num = color;    // no problem

    if (color == Card::red_card) // no problem (bad)
        cout << "bad" << endl;

    if (card == Color::green)   // no problem (bad)
        cout << "bad" << endl;

    // examples of good use of enum classes (safe)
    Animal a = Animal::deer;
    Mammal m = Mammal::deer;

    int num2 = a;   // error
    if (m == a)         // error (good)
        cout << "bad" << endl;

    if (a == Mammal::deer) // error (good)
        cout << "bad" << endl;
```
#### 2. They don't pollute the surrounding namespace.

```cpp
enum class Color1 { red, green, blue };    //this will compile
enum class Color2 { red, green, blue };

enum Color1 { red, green, blue };    //this will not compile 
enum Color2 { red, green, blue };
```

#### 3. They can be forward-declared
```cpp
enum class E_MY_FAVOURITE_FRUITS : unsigned char
{
    E_APPLE        = 0x01,
    E_WATERMELON   = 0x02,
    E_COCONUT      = 0x04,
    E_STRAWBERRY   = 0x08,
    E_CHERRY       = 0x10,
    E_PINEAPPLE    = 0x20,
    E_BANANA       = 0x40,
    E_MANGO        = 0x80,
    E_DEVIL_FRUIT  = 0x100, // Warning!: constant value truncated
};

```

### N2. {} vs = or ()

####  {}: C++11,  prevents narrowing
```cpp
int bad(1.0);  //will compile
int bad{1.0}; //error: narrowing conversion of '1.0e+0' from 'double' to 'int' [-Wnarrowing]
```

https://en.cppreference.com/w/cpp/language/list_initialization
   
#### to gcc 10.2.0 no warnings! (it treats it as declaration of function with void value which returns Test object)
```cpp
   class Test {public: Test(){std::cout<<"constructing"<<std::endl;}};
   Test t1();
   Test t2{};
```   

#### start from gcc 11.1.0:

```cpp
warning: empty parentheses were disambiguated as a function declaration [-Wvexing-parse]
    8 |    Test t1();
      |           ^~
 note: remove parentheses to default-initialize a variable
    8 |    Test t1();
      |           ^~
      |           --
 note: or replace parentheses with braces to value-initialize a variable   
```  

### N3. Cyclomatic Complexity (złożonośc cyklomatyczna)
 Podstawą do wyliczeń jest liczba dróg w schemacie blokowym danego programu, co oznacza wprost liczbę punktów decyzyjnych w tym programie.

Przytaczane są poniższe wartości złożoności cyklomatycznej:
od 1 do 10 – kod dość prosty stwarzający nieznaczne ryzyko
od 11 do 20 – kod złożony powodujący ryzyko na średnim poziomie
od 21 do 50 – kod bardzo złożony związany z wysokim ryzykiem
powyżej 50 – kod niestabilny grożący bardzo wysokim poziomem ryzyka.
https://www.youtube.com/watch?v=TmWc4mdLmvQ&ab_channel=CppNow 

**Tools:**
1. https://people.debian.org/~bame/pmccabe/pmccabe.1 pmccabe
2.  git clone https://github.com/terryyin/lizard.git 

python lizard.py -CCN 10 -w someFileToParese.cppreference

### N4. Short-Circuiting Evaluation in C++ and Linux

Some compiler can allow you to disabled this option like: gcc *-fno-short-circuit-optimize*

**example 1**
```cpp
int main()
{

    // Short circuiting
    // logical "||"(OR)
    int a = 1, b = 1, c = 1;
 
    // a and b are equal
    if (a == b || ++c) {
        std::cout << "True 0" << std::endl;
    }
    else {
        std::cout << "False 0" << std::endl;
    }
 
    // Short circuiting
    // logical "&&"(AND)
 
    if (a != b && ++c) {
        std::cout << "True 1" << std::endl;
    }
    else {
        std::cout << "False 1" << std::endl;
    }
    std::cout << c <<std::endl;
}
```  
**example 2**
```cpp
    // Short circuiting
    int a = 1, b = 1;
 
    auto foo = []() { std::cout<<"function called"<<std::endl; return 1; };
    // a and b are equal
    int d = a == b || foo() ? 1 : 0;
    int f = a != b || foo() ? 1 : 0;

    std::cout << d <<" "<< f<<std::endl;
```  
**example 3**
```sh
[[ -e “$filename” ]] && echo “exists”
```  
### N5. Tooggle bool parameters
Let's imagine we spot the snipped production code like:
```cpp
    doSomething(false, true, true); // the ugliest one
```  

**1 Just comments**

Better than first one but still confused regarding clean code rules
```cpp
    doSomething( /*highSpeed*/ false, /*optimize*/ true, /*debug*/ true );
``` 

**2 Bit flags**
```cpp
enum class OptionModes : uint8_t
{
    highSpeed = 1,
    optimize = 2,
    debug = 4
};

//missing check if the new value is in range...
OptionModes operator | (OptionModes a, OptionModes b) 
{
    using T = std::underlying_type_t <OptionModes>;
    return static_cast<OptionModes>(static_cast<T>(a) | static_cast<T>(b));
}
constexpr bool isFlagSet(OptionModes val, OptionModes check) 
{
    using T = std::underlying_type_t <OptionModes>;
    return static_cast<T>(val) & static_cast<T>(check);
}

void doSomething(OptionModes flags)
{
    if(isFlagSet(flags, OptionModes::highSpeed))
    ...
}

doSomething(OptionModes::highSpeed | OptionModes::debug);
``` 

**3 Enum class**
```cpp
enum class HighSpeed { disabled = false, enabled = true };
enum class Optimize { disabled = false, enabled = true };
enum class Debug { disabled = false, enabled = true };

doSomething(HighSpeed::disabled, Optimize::enabled, Debug::enabled);
```


**4 Param structure**
```cpp
struct OptionModes
{
    bool highSpeed{false};
    bool optimize{false};
    bool debug{false};
};

doSomething(OptionModes{.highSpeed = false, .optimize=true, .debug=true });
doSomething(OptionModes{.highSpeed = true});
```

### N6. Copy elision (TBD)

### C++ libraries:
- State Machine Language https://github.com/boost-ext/sml
