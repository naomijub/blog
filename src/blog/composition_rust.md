# Function Composition and Partial Functions in Rust

|date | Description | Keywords |
|--------- |----------------------------- |---------------------------------------- |
|2024-12-20| Function Composition in Rust |Functional Programming, Rust, Composition|

In my latest post, about [functional programming](./functional_programming.md) I talked about Composition and Higher Order functions, but it was merely a theoretical topic, but today I want to introduce examples of function composition and partial functions in Rust, comparing to other languages. 

## Introduction

This is something that has been in my mind for a while, but recently a friend of mine asked how my project deals with unit testing functions that have *side effects*, and my answer was "Higher order functions and function composition". This is something that I have done to the Java/Clojure project we worked together at Thoughtworks some time ago, and I used it as a reminder to that friend. However, more than a reminder to them, I remembered a really old (2019) [composition Rust project](https://github.com/naomijub/rust-compose) I did:

```rust
fn compose<A, B, C, G, F>(f: F, g: G) -> impl Fn(A) -> C
where
    F: Fn(A) -> B,
    G: Fn(B) -> C,
{
    move |x| g(f(x))
}

fn main() {
    let add_and_multiply = compose(|x| x * 3f32, |x| x + 3f32);
    let divide_and_subtract = compose(|x| x / 3f32, |x| x - 3f32);

    let composed = compose(add_and_multiply, divide_and_subtract);
    println!("Result is {}", composed(20f32));
}
```

But now I want to take my time to explain this code and how it would look now.

## What is Function Composition

Function composition is a powerful concept in programming that allows developers to build complex logic by combining functions. It is core to the functional programming paradigm. It allows developers to combine 2 or more functions producing a new function, using functions as input arguments and return types to other functions. mathematically it means that giver the functions `f(x)` and `g(x)`, their composition, `h(x)` can be expressed as:

```
h(x) = g(f(x))
```

The main purposes of function composition are:
- **Reusability**: Reuse smaller, tested functions to build more complex logic.
- **Readability**: Make code more declarative by abstracting low-level details, giving more appropriate namings to blocks.
- **Testability**: Test smaller components independently, and manipulate complex function creating functions that act like mocks for their specific case.

## Implementing Function Composition in Rust

Rust is not necessarily the easiest language to do function composition. However, it is quite expressive in its robustness to do so. Meaning that we can achieve it through closures and `Fn` traits. Let's first go over a simpler case, given the functions:

```rust
fn duplicate(x: i32) -> i32 {
    x * 2
}

fn square(x: i32) -> i32 {
    x * x
}
```

We want to compose them in a way that we square `x` after `duplicating` it. It could easily be done by calling `square(duplicate(3))`, but this is not actually composition, and makes our life a bit harder to read over a long pipe, so we want to be able to compose it as follows `compose(duplicate, square)`, meaning that we will first `duplicate` and then `square` it. Which means that now we have to understand how the `compose` function works:

```rust
fn compose(f: impl Fn(i32) -> i32, g: impl Fn(i32) -> i32) -> impl Fn(i32) -> i32 {
    move |x| g(f(x))
}
```

Compose receives as argument two function implementations of trait `Fn`, `f` and `g`, both of them receive receive as argument a type `i32` and return a type `i32`, then we define `h(x)` as `g(f(x))`, which can be transformed into a function by applying move the closure `|x| g(f(x))`.

> In the first versions of Rust, this was possible by using the `Box` pointer. If you read Portuguese, you can learn a bit more about this in my book [Programação Funcional e Concorrente em Rust](https://www.casadocodigo.com.br/products/livro-rust-funcional-concorrente).

Now that we have defined the compose function, we can use it to compose the two functions we created [(Rust Playground)](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=cc0bb3f79f79918d22b8c5b5a600e38b):

```rust
fn main() {
    let double_then_square = compose(duplicate, square);
    println!("Anonymous composition: {}", compose(duplicate, square)(3)); // Output: 36
    println!("Named composition:     {}", double_then_square(3));         // Output: 36
}
```

There are two ways of using the result from `compose`:
- Assign it to a variable and use it as a named function, in our case `double_then_square`.
- Call it anonymously with the desired `x` value, as `compose(duplicate, square)(3)`.

Some other use cases that are common in Rust ecosystem:
- **Data Transformation Pipelines**: Transforming streams of data in a structured manner.
- **Middleware in Web Frameworks**: Combining pre-processing and post-processing logic.
- **Chained Operations**: Complex mathematical computations or algorithms.

### Comparing to C++

To those that know that I complain a lot about functional programming in C++, I know it possible achieve it using function pointers, lambda expressions, and higher-order functions from libraries like `<functional>` and it looks a lot like what Rust looks like, we just have to consider that `impl Fn(i32) -> i32` becomes the function pointer `std::function<int(int)>`.

```c++
#include <iostream>
#include <functional>

int duplicate(int x) {
    return x * 2;
}

int square(int x) {
    return x * x;
}

std::function<int(int)> compose(std::function<int(int)> f, std::function<int(int)> g) {
    return [f, g](int x) { return g(f(x)); };
}

int main() {
    auto double_then_square = compose(duplicate, square);
    std::cout << "Anonymous composition: " << compose(duplicate, square)(3) << std::endl; // Output: 36
    std::cout << "Named composition: "     << double_then_square(3)         << std::endl; // Output: 36
    return 0;
}
```

While both Rust and C++ allow function composition, Rust’s type system and functional idioms make the process more expressive and safer. The lack of null pointers and the Option type help avoid runtime errors. Also, we can easily make the compose function more generic with generics, which is not as easy in C++:

```rust
fn compose<T>(f: impl Fn(T) -> T, g: impl Fn(T) -> T) -> impl Fn(T) -> T {
    move |x| g(f(x))
}
```

## Partial Functions in Rust

Partial functions are somewhat similar to function composition, but we define a function to a subset of the possible values that it can receive, as if it was using a constant value instead of a function. In this case, I want to extrapolate partial functions from Clojure's [`partial`](https://clojuredocs.org/clojure.core/partial) function.

Clojure's `partial` is a core language utility that allows to pre-fill arguments to a function, creating a new function with fewer parameters. Could say that it is quite common for testing `Dal` like structures that require a lot of configuration. So let's consider the following example:

```clojure
(defn multiply [a b]
  (* a b))

(def multiply-by-three (partial multiply 3))

(println (multiply-by-three 4)) ; Output: 12
```

We have the function `multiply`, but we only care about the case where multiply has the first element set to `3`, binding it as `multiply-by-three` from `(partial multiply 3)`, then we can just call it `(multiply-by-three 4)`. The caveat from Clojure side is that partial can only be applied to the first `n` arguments, while Rust gives you a bit more control over that.

The following [Rust code](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=9dd36547e1566b5e232e405555591d87) is a translation of the previous Clojure code with a twist of having the partial applied to the second argument, demonstrating how Rust enables a bit more control over this:

```rust
fn multiply(a: i32, b: i32) -> i32 {
    a * b
}

fn partial_multiply(b: i32) -> impl Fn(i32) -> i32 {
    move |a| multiply(a, b)
}

fn main() {
    let multiply_by_four = partial_multiply(4);
    println!("Result: {}", multiply_by_four(3)); // Output: 12
}
```

`multiply` function is exactly the same. However, the `partial` application of multiply, shifts which argument we are applying the partiality, as we create `multiply_by_four` applying `partial_multiply` to `b`, instead of `a`. Note that we can apply `multiply_by_four` multiple times and always having the guarantee that we will multiply by `4`.

Use Cases of Partial Functions
- **Pre-binding Arguments**: Useful when testing functions that require a lot of configuration, or event-driven architectures where some functions take repetitive arguments over and over.
- **Currying Simulation**: Breaking down multi-argument functions into simpler, single-argument functions.
- **Code Reusability**: Creating specific versions of generic functions, useful for simplifying test cases as well.