*Last Updated 1/28*
# Applicative vs. Imperative

Most programming languages you have seen so far are **Imperative**. It comes from Latin *Imperator*, meaning something along the lines of *emperor*. Since emperors often gave orders such as "do this" and "do that", imperative languages work similarly. They issue commands to change variables. Most common languages (C, Java, etc) are imperative by design.

**Applicative** languages compute values by calling functions. In these languages, pretty much everything is a function. Rather than mutable variables, program entities are represented by **values bound to names** and **compositions of function calls**. While initially unintuitive, without variables:
1. Problems can be reduced to simpler form because recursive definitions mirror mathematical definitions directly.
2. Common imperative issues and bugs can be eliminated because pure functions avoid shared mutable state, aliasing, and order-dependent behavior.

## C vs. OCaml
The factorial function is defined as:

C *(Imperative)*
```c
int fac(int n) {
	int f = 1; // product to be computed
	int t = n; // counting var
	while(t > 0) {
		f = f * t; // accumulate the product
		t = t - 1; // iterate down
	}
	return f;
}
```

- **Explicit types**
	`int fac(int n)`, `int f`, `int t`, types must be written *everywhere* and are checked statically *(at compile time)*.
	
- **Variables**
	`f` and `t` represent *locations in memory* whose contents change over time.
	
- **Statement language**
	Lines like `f = f * t` and `t = t - 1` are commands that act on values. You can't evaluate them, they directly act on entities.
	
- **Variable assignment/mutation**
	`f = 1`, `f = f * t`, `t = 1 - 1`, each assignment *overwrites* the previous state.
	
- **Return statement converts expression to value**
	`return f` establishes that the function computes effects, then extracts a value.
	
- **Executed Sequentially**
	The meaning of `f = f * t` depends on _when_ it executes and the current value of `f` and `t`.  You must reason about the order of steps to understand correctness.

OCaml *(Applicative)*
```OCaml
let rec facing f t =            (* let +recursive +name +param *)
	if t > 0                    (* if 0, just return           *)
	then facing (f * t) (t - 1) (* call again with new params  *)
	else f;;                    (* ;; means return             *)
let fac n =                     
	facing 1 n ;;               (* call facing with param 1, n *)
```

- **Imperative variables turn into applicative parameters**  
	`f` and `t` are parameters, not mutable storage. Each recursive call gets *new bindings*:  `facing (f * t) (t - 1)` does not modify `f` or `t`; it creates new values.
    
- **No return statement**  
    The value of a function is the value of its final expression. In `else f`, `f` _is_ the result.  `;;` just ends the definition at the top level.
    
- **Expression language**  
    `if t > 0 then ... else ...` is an expression with a value. Everything here evaluates to some expression.
    
- **Loops become recursion**  
    The `while` loop is replaced by a recursive function call. The repetition is expressed structurally, not temporally
    
- **No mutable variables**  
    No assignments (`=` is binding, not mutation). Once `f` and `t` are bound, they never change.
    
- **Function calls don’t use `()` or `,`**  
	`facing 1 n`, not `facing(1, n)`. Application is whitespace-based and left-associative.
    
- **Define before use**  
	`facing` must be defined before `fac` uses it. Same constraint as C at the top level.
    
- **Implicit typing**  
    No types written, but OCaml infers:
    `facing : int -> int -> int` 
    `fac    : int -> int`
    
- **No need to reason about time**
    The function describes _what_ factorial is, not _how to mutate state to compute it_. Evaluation order matters for performance, but not for correctness

> OCaml **can** write imperative code (mutable refs, arrays, loops), but the language _defaults_ to applicative reasoning.

# How to talk to OCaml

OCaml has a **top-level** (the REPL), usually started with:

```bash
ocaml
```

This is *not* just a shell that runs programs. It’s an **interactive evaluator**.
When you type something and end it with `;;`, OCaml:
1. Parses it
2. Type-checks it
3. Evaluates it
4. Prints the result and its type

```ocaml
# 2 + 2 ;;
- : int = 4
```
- `-` means “this expression wasn’t bound to a name”
- `int` is the inferred type
- `4` is the value

In OCaml, **everything is an expression**, and expressions always have a value and a type.
## Expression Statements

```ocaml
let n = 4 ;;
val n : int = 4
```

This is not “assigning 4 to a variable” in the imperative sense. It is closer to “From now on, `n` refers to the value `4`.”

There is no mutation, no memory cell being updated, no reassignment (unless you explicitly use mutable constructs). You can think of `let` as naming a value, expression, or function.

Functions are also one-input, one-output.

```ocaml
val fac : int -> int = <fun>
```

That arrow means “A function that takes an `int` and returns an `int`."
## Call notation
OCaml also uses different function call notation:

```ocaml
add 2 3
```

Parses as:
```ocaml
((add 2) 3)
```

## Recursion is explicit
OCaml does not assume functions can call themselves. You must say use `rec`:

```ocaml
let rec fac n =
  if n = 0 then 1 else n * fac (n - 1) ;;
```

# Lists

OCaml lists are:
- **Immutable** (cannot change, only copy)
- **Singly linked** (represented by a linked list)
- **Homogenous** (all elements of the same type)
Theire expression typing looks like:
```OCaml
int list
string list
'a list (* polymorphic list *)
```
Internally, they're either empty, or an element + another list.

```Ocaml
type 'a list =
  | []
  | (::) of 'a * 'a list
```
A list is basically `head :: tail`. The head is the first element, the tail is the rest of the list.

`::` **prepends** an element to a list.
```OCaml
1 :: [2;3] (* [1; 2; 3] *)
```

The operator is an expression, and has a type `(::) : 'a -> 'a list -> 'a list`. Because of the formating of `head :: tail`, all lists are just cons operations on an empty list: `[1;2;3] = 1 :: 2 :: 3 :: []`.

## Pattern Matching
You can use `match lst with` to "if-else" list contents:
```OCaml
match lst with
| [] -> ...
| h :: t -> ...
```
*`h` and `t` are stylistic choices. The left operator will refer to the head, the right will refer to the tail*.

There are also built-ins, but they aren't as used:
```Ocaml
List.hd lst
List.tl lst
```
*These throw exceptions on `[]`, and are discouraged because of that.*

Match list also allows for convenient recursion, since the empty list will be your base case and the other will be your recursive trigger:
```OCaml
let rec sum lst =
  match lst with
  | [] -> 0 (*base case*)
  | h :: t -> h + sum t (*recursive call*)
```

If you aren't using one of the operators in the return call, a `_` takes the place of a head/tail reference:
```Ocaml
let rec length = function
  | [] -> 0
  | _ :: t -> 1 + length t
```
*`function` is another syntax shortcut for `match`.*

## Appending
You can append lists using a native operator, but it is slower since it walks the entire left list:
```Ocaml
lst1 @ lst2
```

*List append function*
```OCaml
let rec append l r =
	if r = [] then l
	else if l = [] then r
	else (hd l) :: (append (tl l) r) ;;
	
(*
'a list -> 'a list -> 'a list
append [1;2] [3;4]
1 :: (append [2] [3;4])
1 :: (2 :: (append [] [3;4]))
1 :: (2 :: [3;4])
1 :: [2;3;4]
[1;2;3;4]
*)
```

# OCaml functions
## Multiple Arguments
OCaml thinksthat all functions take 1 argument t1 and return 1 value t2. The type of the function is then t1->t2. To model a function that takes in more than one argument, every function type before the last is an argument. 

```
t1 -> t2 -> t3 -> v (return value)
'---v---'
argument types
```

For everything inbetween, a function of 1 argument is a function that returns another function.

## List terminology
**First class objects** can be passed as an argument, return it from a function, assign it to a 'variable'. And make it part of a data structure. These are all properties of functions in applicative languages. **Second class objects** are just all objects that aren't first class.

Lists in OCaml are **immutable**, meaning we cannot change the list directly. So list operations with multiple lists must:
- Copy at least part of a list
- Return a copy of the list operation result

**Persistence means that** when we operate on two or more data structures (like lists), the original structures are unchanged.

## Internal Helper Functions




## Tail Recursion



Types  
Pattern matching  
Expressions  
Environments  
Closures  
Curried functions  
Higher-order functions  
Continuation passing  
Generators  
Objects with functions as parts  
Object oriented programming  
Memoization  
Evaluation strategies  
Eager vs. lazy evaluation  
Organizing large programs  
Modules  
Signatures  
Lisp  
Grammars  
Scanners  
Parsers  
Interpreters  
More interpreters  
Metaprogramming  
Macros
