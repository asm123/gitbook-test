# C++: Template Metaprogramming

Recently, I heard the term "template metaprogramming" from someone at work. I had no clue about what it was. So I started reading about it. These are the notes from some very light reading.

### Templates in C++

> References
>
> * [cppreference](https://en.cppreference.com/w/cpp/language/templates)
> * [Wikipedia](https://en.wikipedia.org/wiki/Template_\(C%2B%2B\))
> * [Microsoft Learn](https://learn.microsoft.com/en-us/cpp/cpp/templates-cpp)

Template is a parameterized construct that generates a type or a function at _compile_ time based on the arguments provided by the user for the template parameters.

A template can be one of the following:

*   **function template** - behaves like a function, template can have arguments of many different types

    Syntax:

    ```cpp
    // With typename
    template <typename identifier> declaration;
    // With class - behaves same as typename
    template <class identifier> declaration;
    ```

    Example:

    ```cpp
    template <typename T>
    int min(T a, T b) {
      return a < b ? a : b;
    }

    int main() {
      // T = int, automatically deduced
      std::cout << std::min(3, 5) << std::endl;
      // T = double, automatically deduced
      std::cout << std::min(3.0, 5.0) << std::endl;
      // T = double, cannot be deduced automatically, 
      // needs to be mentioned explicitly
      std::cout << std::min<double>(3.0, 5) << std::endl;
    }
    ```
*   **class template** - provides specification for generating classes based on parameters

    Syntax:

    ```cpp
    template <class T>
    class className {
      T methodName(T arg);
    };
    ```

    Example:

    ```cpp
    template <class K, class V> 
    class Map {
      V getValue(K k);
    };
    ```

According to [cppreference.com](https://en.cppreference.com/w/cpp/language/templates), there are a few more types of templates. Understanding function and class templates suffice for now.

### Template specializations

Defining special behavior of a template for a particular data type is template specialization.

* Full specialization: filling in all the parameters for the template.
* Partial specialization: defining special behavior for only some of the parameters of the template.

Example:

```cpp
// Template definition
template <typename K, typename V>
class Map {
  Map() {
    std::cout << "Map" << std::endl;
  }
}

// Partial specialization
template<int, typename V>
class Map<int, V> {
  Map() {
    std::cout << "Map<int, V>" << std::endl;
  }
}

// Full specialization
template<int, std::string V>
class Map<int, std::string> {
  Map() {
    std::cout << "Map<int, std::string>" << std::endl;
  }
}

// Use
int main() {
  auto defined = Map<float, bool>{}; // Map
  auto partial = Map<int, bool>{}; // Map<int, V>
  auto full = Map<int, std::string>{}; // Map<int, std::string>
}
```

### Metaprogramming

According to [Wikipedia](https://en.wikipedia.org/wiki/Metaprogramming),

> Metaprogramming is a computer programming technique in which computer programs have the ability to treat other programs as their data. It means that a program can be designed to read, generate, analyse, or transform other programs, and even modify itself, while running.

Wait what? A program can even modify itself while running? According to [this StackOverflow answer](https://stackoverflow.com/a/42220709), that is the simplest definition of metaprogramming! Moreover, according to the answer,

> Your accepted answer says programs that manipulate themselves. Those are indeed metaprograms but they are a subset of all metaprograms.
>
> All:
>
> * Parsers
> * Domain specific languages (DSLs)
> * Embedded domain specific languages (EDSLs)
> * Compilers
> * Interpreters
> * Term rewriters
> * Theorem provers
>
> are metaprograms.

* Programming transforms data, metaprogramming transforms programs to code. A metaprogram that looks at its own structure is Reflection.

[This StackOverflow answer](https://stackoverflow.com/questions/980492/what-is-metaprogramming) explains it with a real-world example.

* More commonly, metaprogramming is used for types rather than values.
  * We are manipulating types, not values.
  * It is used to facilitate generic programming, e.g. [iterators](https://gcc.gnu.org/onlinedocs/libstdc++/libstdc++-api-4.5/a00906_source.html).

### Template metaprograms

#### if-else

> Reference: [Todd Veldhuizen](https://www.cs.rpi.edu/~musser/design/blitz/meta-art.html)

```cpp
template <bool C>
class _name {};

// true case
class _name<true> {
  public:
    static inline void f() { statement1; }
};

// false case
class _name<false> {
  public:
    static inline void f() { statement2; } // false
};

// call
_name<condition>::f();
```

Remember _compile-time_! The `condition` must be known at compile-time.

#### for loop

> Reference: [Niko Savas](https://medium.com/@savas/template-metaprogramming-compile-time-loops-over-class-methods-a243dc346122)

```cpp
// Template definition
template <int N>
void forLoop() {
  std::cout << N << std::endl;
  
  forLoop<N-1>();
}

// Partial specialization for the end of loop
template<>
void forLoop<0>() {
  // Loop has finished, do nothing
  std::cout << "Done!" << std::endl;
}
```

### When to use?

* Code needs to be generic
* Computation needs to be at compile-time for better run-time performance

### Further study

* [BitsOfQ playlist on C++ template metaprogramming](https://youtube.com/playlist?list=PLWxziGKTUvQFIsbbFcTZz7jOT4TMGnZBh\&feature=shared)
* TODO: Add some code snippets for relatable examples.
