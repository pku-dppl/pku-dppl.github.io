---
layout: default
title: Projects
---

# Projects

## Project Requirement

In this course, you will design and implement a typed programming language with certain features.
You are encouraged to draw inspiration from popular or emerging languages, such as Rust, Go, TypeScript, Elm, Scala, Haskell, OCaml, MoonBit, Koka, Crystal, and Zig.
As a course project, you need to choose a key feature that interests you, formalize a core calculus for it (ideally based on lambda calculus), prove its soundness (at least roughly), and implement a prototype (ideally based on the checkers [here](https://github.com/pku-dppl/TAPL-in-MoonBit/) from our course material) with an interpreter and a type checker.
At the end of the semester, you will give a presentation about your work and submit an artifact containing the following:
- A document that includes your motivation, illustrative examples, a formalization of the core calculus, and a soundness proof.
- A prototype implementation of your language along with a suite of benchmark programs.

Below are several potential projects you might consider.
You are also encouraged to come up with your own project!

Table of Content:
- [Project 1: Extensible Records](#project-1-extensible-records)
- [Project 2: Gradual Typing](#project-2-gradual-typing)
- [Project 3: Typeclass or Trait](#project-3-typeclass-or-trait)
- [Project 4: Functional In-place Update](#project-4-functional-in-place-update)
- [Project 5: Refinement Types](#project-5-refinement-types)
- [Project 6: Asynchronous Programming](#project-6-asynchronous-programming)

## Project 1: Extensible Records

In Elm, records are full-fledged values, i.e., one does not need to declare a type for records.
Programmers can manipulate records in a very flexible way; see the examples below.

``` elm
point2D = { x = 0, y = 0 }
point3D = { x = 3, y = 4, z = 12 }
points = [point2D, point3D]

point3D.z           -- 12
.y point3D          -- 4
List.map .x points  -- [0, 3]

{ point2D | y = 1 }         -- { x = 0, y = 1 }
{ point3D | x = 0, y = 0 }  -- { x = 0, y = 0, z = 12 }

hypotenuse {x, y} = sqrt (x^2 + y^2)
hypotenuse point2D  -- 0
hypotenuse point3D  -- 5
```

The last example with `hypotenuse` demonstrates *row polymorphism*, i.e., the function accepts any records with at least two fields `x` and `y`.
This could be achieved by subtyping, but this project aims at a more powerful mechanism for *extensible records*, i.e., field addition and deletion; see the examples below.

``` elm
{ point3D - z }      -- { x = 3, y = 4 }
{ point2D | z = 0 }  -- { x = 0, y = 0, z = 0 }
```

Please refer to the [Leijen 2005] paper to see how to use polymorphism to support extensible records.

Reference: Daan Leijen. 2005. Extensible records with scoped labels. In *Trends in Functional Programming (TFP'05)*.

Starter: [System F](https://github.com/pku-dppl/TAPL-in-MoonBit/tree/main/chap23-fullpoly)

Difficulty: ★

## Project 2: Gradual Typing

TypeScript features `any` types to turn off type checker wherever they appear; such types are designed to bridge static and dynamic typing.
See the examples below.

``` typescript
// static: error
var answer : number = 42;
var the_answer : string = answer;

// static: ok, dynamic: ok
var answer : string = "42";
var a : any = answer;
var the_answer : string = a;

// static: ok, dynamic: error
var answer : number = 42;
var a : any = answer;
var the_answer : string = a;
```

Things become more interesting when `any` interacts with functions:

``` typescript
// static: ok, dynamic: ok
function f(x : string) : string { return x; }
var g : (any) => string = f;
var h : any = g;
var x : string = h("42");

// static: ok, dynamic: error
function display(y : string) {
  console.log(y.charAt(0)); // error is raised at the charAt call
}
var x : any = 3;
display(x);
```

You may note that although the examples work as expected, the last one is not satisfactory because the runtime error is raised too late: a more ideal place to raise the error is at the `display` call.
This is called the *blame tracking* problem, and you may try to come up with a better mechanism to assign blames in this project.

Please refer to the [Siek and Taha 2006] paper to see how to formalize the idea of gradual typing.

Reference: Jeremy G. Siek and Walid Taha. 2006. Gradual Typing for Functional Languages. In *Scheme and Functional Programming Workshop*.

Starter: [Simply-Typed Lambda Calculus](https://github.com/pku-dppl/TAPL-in-MoonBit/tree/main/chap11-fullsimple)

Difficulty: ★

## Project 3: Typeclass or Trait

You are familiar with C++'s operator overloading.
You may also be familiar with Haskell's typeclasses or Rust/MoonBit's traits.
No matter how people call them, they all provide *ad-hoc polymorphism*, i.e., a function (or a set of functions) can be defined over several different types, acting in a possibly different way for each type.
See the following Rust example.

``` rust
pub trait Summary {
  fn summarize(&self) -> String;
}

pub struct NewsArticle {
  pub headline: String,
  pub location: String,
  pub author: String,
  pub content: String,
}

impl Summary for NewsArticle {
  fn summarize(&self) -> String {
    format!("{}, by {} ({})", self.headline, self.author, self.location)
  }
}

pub struct Tweet {
  pub username: String,
  pub content: String,
  pub reply: bool,
  pub retweet: bool,
}

impl Summary for Tweet {
  fn summarize(&self) -> String {
    format!("{}: {}", self.username, self.content)
  }
}
```

In this project, you are supposed to design a language that combines first-class polymorphism and ad-hoc polymorphism.
Please refer to the [Odersky et al. 1995] paper to see how to formulate typeclasses/traits in a lambda calculus.

Reference: Martin Odersky, Philip Wadler, and Martin Wehr. 1995. A Second Look at Overloading. In *Functional Programming Languages and Computer Architecture (FPCA'95)*.

Starter: [System F](https://github.com/pku-dppl/TAPL-in-MoonBit/tree/main/chap23-fullpoly)

Difficulty: ★★

## Project 4: Functional In-place Update

One of the concepts behind Rust is *linear typing*: an informal description is that every program variable can be used once, so that essentially every variable owns its data and using it basically moves the data to elsewhere.
It has been shown that linear typing can serve as a "design pattern" for writing `malloc`-free C code in a functional style; see the examples below.

``` c
typedef enum {NIL, CONS} kind_t;
typedef struct lnode {
  kind_t kind;
  int hd;
  struct lnode * tl;
} list_t;

typedef void * dia_t;

list_t nil(){
  list_t res;
  res.kind = NIL;
  return res;
}

list_t cons(dia_t d, int hd, list_t tl){
  list t res;
  res.kind = CONS;
  res.hd = hd;
  *(list_t *)d = tl;
  res.tl = (list_t *)d;
  return res;
}
```

The type `dia_t` implements the idea of linear typing: if you want to create a heap cell (e.g., in `cons`), you must move in a `dia_t`.
With some tricks, we can then manipulate lists in a functional style:

``` c
typedef struct {
  kind_t kind;
  dia_t d;
  int hd;
  list_t tl;
} list_destr t;

list_destr_t list_destr(list_t l) {
  list_destr_t res;
  res.kind = l.kind;
  if (res.kind == CONS) {
    res.hd = l.hd;
    res.d = (void *) l.tl;
    res.tl = *l.tl;
  }
  return res;
}

list_t append(list_t l1, list_t l2) {
  list_destr_t ld = list_destr(l1);
  return ld.kind == NIL ? l2 : cons(ld.d, ld.hd, append(ld.tl, l2));
}
```

In this project, you are supposed to formulate a linear functional language to capture the pattern of functional in-place update.
Please refer to the [Hofmann 2000] paper to see how to develop a linearly-typed lambda calculus.

Reference: Martin Hofmann. 2000. A type system for bounded space and functional in-place update. In *Nordic Journal of Computing 7(4)*.

Starter: [Simply-Typed Lambda Calculus](https://github.com/pku-dppl/TAPL-in-MoonBit/tree/main/chap11-fullsimple)

Difficulty: ★★

## Project 5: Refinement Types

LiquidHaskell refines Haskell's types with *logical predicates* that let you enforce important properties at compile time; see the examples below.

``` haskell
{-@ predicate NonNull X = ((len X) > 0) @-}
{-@ head :: {v:[a] | (NonNull v)} -> a @-}
head (x:_) = x

{-@ dotProduct :: x:(Vector Int)
               -> y:{v: (Vector Int) | (vlen v) = (vlen x)}
               -> Int
  @-}
dotProduct x y = sum [ (x ! i) * (y ! i) | i <- [0 .. n - 1]]
  where
    n = length x

{-@ type IncrList a = [a]<{\xi xj -> xi <= xj}> @-}
{-@ insertSort :: (Ord a) => xs:[a] -> (IncrList a) @-}
insertSort []     = []
insertSort (x:xs) = insert x (insertSort xs)
```

The first example shows how to guarantee functions are total. The second example shows how to keep array/vector/... indexes within bounds.
The third example shows how to enforce correctness properties.

In this project, you are supposed to design a constraint-based type system to encode and check such logical constraints.
You may need to choose an underlying logic (e.g., propositional logic with linear integer arithmetics) and implement a solver for it (or find some third-party library).
As for the formalization, please refer to the [Rondon et al. 2008]'s paper about liquid types.

Reference: Patrick M. Rondon, Ming Kawaguchi, and Ranjit Jhala. 2008. Liquid Types. In *Programming Language Design and Implementation (PLDI'08)*.

Starter: [System F](https://github.com/pku-dppl/TAPL-in-MoonBit/tree/main/chap23-fullpoly)

Difficulty: ★★★

## Project 6: Asynchronous Programming

Many modern languages now support asynchronous programming, a technique that allows programs to run other tasks while waiting for a long-running task to finish.
It becomes particularly useful when engineering AI agents, where a request to large models may take a long time.
See the C# example below.

``` c#
static async Task Main(string[] args) {
  Console.WriteLine("Let's start ...");
  var done = DoneAsync();
  Console.WriteLine("Done is running ...");
  Console.WriteLine(await done);
}

static async Task<int> DoneAsync() {
  Console.WriteLine("Warming up ...");
  await Task.Delay(3000);
  Console.WriteLine("Done ...");
  return 1;
}
```

The `Main` function calls `DoneAsync` *asynchronously* in the sense that it does not wait for the result of `DoneAsync`.
Only until the last line, the `await done` waits for the completion of `DoneAsync`.
There are possibly many different way to formalize asynchronous programming, but here we suggest using *effects*.
Koka is a programming language that supports algebraic effects, so one can implement an `async` effect and use it together with other effects.
Below is an example with a `yield` effect, which is not exactly asynchronous but demonstrates the pattern of coroutines.

```
// A generator effect with one operation
effect yield<a>
  fun yield( x : a ) : ()

// Traverse a list and yield the elements
fun traverse( xs : list<a> ) : yield<a> ()
  match xs
    Cons(x,xx) -> { yield(x); traverse(xx) }
    Nil        -> ()

fun main() : console ()
  with fun yield(i : int)
    println("yielded " ++ i.show)
  [1,2,3].traverse
```

In this project, you do not need to consider a full system of algebraic effects.
You are supposed to integrate lambda calculus with some effects sufficient for asynchronous programming.
Please refer to the [Leijen 2017] paper to see a detailed presentation of algebraic effects and asynchronous programming.

Reference: Daan Leijen. 2017. Structured Asynchrony with Algebraic Effects. In *Type-Driven Development (TyDe'17)*.

Starter: [Simply-Typed Lambda Calculus](https://github.com/pku-dppl/TAPL-in-MoonBit/tree/main/chap11-fullsimple)

Difficulty: ★★★
