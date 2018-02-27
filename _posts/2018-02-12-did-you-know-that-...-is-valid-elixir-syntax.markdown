---
layout: post
title:  "Did you know that \"............\" is valid Elixir syntax?"
date:   2018-02-12 17:11:33 +0100
categories: elixir
---
This is working code, running in IEx, with just one module imported. Can you guess the definition of the module?

```elixir
iex(1)> ...............
42
```

We can even add more dots or remove some:

```elixir
iex(1)> ...........................
42
iex(2)> ......
42
```

To make the code above work we need the following module definition and `import` the module we defined:

```elixir
iex(1)> defmodule MyModule do
...(1)>   def ...... do
...(1)>     ...
...(1)>   end
...(1)>
...(1)>   def ... do
...(1)>     42
...(1)>   end
...(1)> end
{:module, MyModule,
 <<70, 79, 82, 49, 0, 0, 4, 44, 66, 69, 65, 77, 65, 116, 85, 56, 0, 0, 0, 126,
   0, 0, 0, 13, 15, 69, 108, 105, 120, 105, 114, 46, 77, 121, 77, 111, 100, 117,
   108, 101, 8, 95, 95, 105, 110, 102, 111, ...>>, {:..., 0}}
iex(2)> import MyModule
MyModule
iex(3)> ......
42
```

It works because Elixir has optional parenthesis and `...` is a valid identifier. To understand it we can look at a simpler expression `......`. If we print the code with `Macro.to_string/1` it is easier to see what is going on:

```elixir
iex(1)> Macro.to_string(quote(do: ......))
"...(...)"
```

Now we can see it's a function call to the `.../1` function passing in the identifier `...` as argument. When looking at the code, without context, `...` can either be a call to the function `.../0` or a reference to the variable `...`. When there is no variable in context with that name the compiler assumes it's a function call which makes the code with all syntax sugar removed: `...(...())`. To make it clear as day we can replace `...` with `foo`:


```elixir
defmodule MyModule do
  def foo(foo) do
    foo
  end

  def foo() do
    42
  end
end
```

But why is `......` parsed as two `...` identifiers, it could also be parsed as the `.` operator, for remote function calls or struct field reference, or the `..` operator, used for ranges. This is an ambiguity in the tokenization process of the compiler—which means there are multiple possible interpretations of a piece of code.

There are actually a lot of other ambiguities in the tokenization of operators, for example using the unary operators `+` and `-` we can build this valid expression `+-1`, but using `+` twice `++1` would parse as the binary list concatenation operator `++` and would fail to compile. To be as consistent as possible, the compiler tries to parse longer symbol tokens before shorter ones. This means the binary syntax operators `<<` and `>>` will parse with higher priority than the comparison operators `<` and `>`, `++` with higher priority than `+`, and `...` with higher priority than `..` and `.`. If we look in the [tokenization module](https://github.com/elixir-lang/elixir/blob/4b134c8cf56811c1f7e9e1a1a053bc22b6d1d5ac/lib/elixir/src/elixir_tokenizer.erl#L296) we can see that `...` has its clause listed first of all operator tokens and longer tokens have their clauses listed before shorter ones.
