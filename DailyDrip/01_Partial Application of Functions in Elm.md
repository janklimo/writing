# Partial Application of Functions in Elm
A friend of mine recently raised an interesting question. _If a function
takes two arguments, why is its type annotation `f: a -> b-> c`? Wouldn't
something like `f: (a, b) -> c` make more sense?_

A solid understanding of how Elm treats functions will make the answer clear
and allow us to write better code.

#### One argument at a time
Let's take the following example:

```elm
greeting : String -> String -> String
greeting hello name = hello ++ ", " ++ name ++ "!"
```

On the surface, this looks like a function that takes two arguments and
returns a result:

```elm
greeting "Hello" "DailyDrip" == "Hello, DailyDrip!"
```

However, functions in Elm always take *exactly* one argument and return
a result (hence the function type annotation syntax).

Passing a single argument to our `greeting` function returns a new function
that takes one argument and returns a string (commented-out output in
the examples below comes from `elm-repl`):

```elm
helloGreeting = greeting "Hello"
-- <function> : String -> String
```

Passing an additional argument to this partially applied function will
yield the expected result:

```elm
helloGreeting "DailyDrip"
-- "Hello, DailyDrip!" : String
```

This shows that the following notation is equivalent:

```elm
greeting "Hello" "DailyDrip" == ((greeting "Hello") "DailyDrip")
```

Parentheses are optional because function evaluation associates to the
left by default. Let's take a look at some examples where partial
application is especially useful.

#### Expressive function definitions
Let's begin with a simple example of a function that doubles a number.
Our first take could be:

```elm
double n = n * 2
-- <function> : number -> number
```

The beautiful thing about Elm is that even operators are functions:

```elm
(*)
-- <function> : number -> number -> number
```

We can use partial application to be more concise while achieving
the same result:

```elm
double = (*) 2
-- <function> : number -> number
```

Because `(*)` takes two numbers as arguments, the compiler will infer that
our `double` function takes one number as its sole argument.

While a function to double all elements of a list could be written as:

```elm
doubleList list = List.map double list
-- <function> : List number -> List number
```

by applying the same principle, we can do better:

```elm
doubleList = List.map double
-- <function> : List number -> List number
```

#### Piping
Now that we know that operators are functions too, we can fully
grasp how piping works:

```elm
(|>)
-- <function> : a -> (a -> b) -> b
```

It really is no magic: the second argument is a function, which makes it
one of the most common use cases for partial application. A good example to
demonstrate it is the following function that returns the sum of all deposits:

```elm
amountDeposited : List Transaction -> Float
amountDeposited list =
    List.filter (\t -> t.type_ == Deposit) list
        |> List.map .amount
        |> List.sum
    -- rather than the nested equivalent:
    -- List.sum (List.map .amount (List.filter (\t -> t.type_ == Deposit) list))
```

There's one important design feature of Elm that makes all of this possible:
[data structure is always the last argument](http://package.elm-lang.org/help/design-guidelines#the-data-structure-is-always-the-last-argument).
This makes writing better, more idiomatic Elm code using piping
and partial application a breeze.
