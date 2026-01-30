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
An **internal helper function** is a function defined _inside_ another function using `let … in`. It exists only within the scope of the enclosing function and is typically used to support the main function’s logic without exposing extra parameters or implementation details.

Because OCaml values **immutability and recursion**, many algorithms that would normally rely on loops or mutable variables instead require extra parameters (such as counters or accumulators). Helper functions allow these parameters to exist internally without cluttering the main function’s signature.

## Basic structure and scope
A helper function is introduced with `let` and made available using `in`. It has access to all variables in the enclosing scope (lexical scoping), but nothing outside can access it.

```ocaml
let f x =
  let helper y =
    y + x
  in
  helper 3
```

Here, `helper` can use `x`, but callers of `f` cannot call `helper` directly.

## Helpers and recursion
When a helper function needs to call itself, it must be declared with `let rec`. This is extremely common, especially for list processing.

The outer function is often _not_ recursive; the helper does the recursive work.

```ocaml
let sum lst =
  let rec aux acc lst =
    match lst with
    | [] -> acc
    | h :: t -> aux (acc + h) t
  in
  aux 0 lst
```

In this pattern:
- `sum` is the public function    
- `aux` is the internal recursive worker    
- `acc` carries intermediate state

## Tail recursion motivation
Many naive recursive functions are not tail-recursive and can cause stack growth. Helper functions allow you to rewrite these using accumulators so the recursive call is the final operation.

This is not just an optimization detail — in many OCaml courses, **tail recursion is a requirement**, and helper functions are the expected solution.

## Accumulators as implementation detail
Accumulators represent “state” that would otherwise be mutable in imperative code. These variables should generally _not_ appear in the function’s public interface.

Helper functions let you introduce them locally, reinforcing that they are part of the algorithm, not part of how the function is used.

## Lexical scoping and captured variables
Helper functions automatically capture variables from the enclosing function. This avoids unnecessary parameter passing and keeps code concise.

```ocaml
let scale_and_sum factor lst =
  let rec aux acc lst =
    match lst with
    | [] -> acc
    | h :: t -> aux (acc + factor * h) t
  in
  aux 0 lst
```

`factor` is shared implicitly, safely, and immutably.

## Multiple helpers
You can define multiple internal helper functions, each handling a distinct responsibility. This is useful for clarity or when breaking a problem into conceptual sub-steps.

Helpers can also call each other, including via mutual recursion, while remaining fully encapsulated within the outer function.

## Exam Maybe
You are expected to use internal helpers when:
- tail recursion is mentioned
- accumulators are needed
- mutation is forbidden
- the desired function signature is simple but the logic is not
# Imperative vs Applicative Loops

## Stack Review
All languages maintain a **stack**, that space in memory that grows downwards. The stack supports push and pop operations to add and remove items from the top. Functions are stored on the stack as **frames**. A frame contains parameters, local names, and the return point.
- Everytime we call a function, we *push* a new frame on the stack.
- Everytime we return from a function, we *pop* a frame off the stack.

The topmost frame of the stack is the currently executing function. We draw them "upside down" because technically, they fall towards lower addressess:

```
         ┌─────────────┐
High Addr│Stack        │
         ├──────┬──────┤
         │      ▼      │
         │             │
         │             │
         │      ▲      │
         ├──────┴──────┤
Low Addr │Heap         │
         └─────────────┘
```

## Factorial Functions (again)

*Remembering our Imperative factorial:*
```c
int fac(int n) {
	int f = 1;
	while (n > 0) {
		f = n + f;
		n = n - 1;
	}
	return f;
}

int main() {
	int k = fac(s)
}
```

In this case, the stack frame for this function makes a copy of `n`, as well as stores `f`. It also stores the address of `k`, the variable `fac()` goes into. After it's done executing and returns, the one frame is popped off the stack.

*Remembering our Applicative factorial:*
```Ocaml
let rec fac n =
	if n = 0 then 1
	else n * fac(n-1) ;;
	
let k = fac 3 ;;
```

In this case, the below diagram occurs, where each box represents a stack frame and it's contents:
```    
┌─────────────┐      
│n = 3        │      
│Return addr k│      
│n * ____  ◄──┼───┐  
└─────────────┘   │  
┌─────────────┐   │  
│n = 2        │   │  
│Return addr ─┼───┘  
│n * ____  ◄──┼───┐  
└─────────────┘   │  
┌─────────────┐   │  
│n = 1        │   │  
│Return addr ─┼───┘  
│n * ____  ◄──┼───┐  
└─────────────┘   │  
┌─────────────┐   │  
│n = 0        │   │  
│Return 1 ────┼───┘      
└─────────────┘
```

Here, ==the depth of the stack is proportional to the number of recursive calls==, and alll the computation ocurs as we pop frames off and move up the stack. The function calls are waiting to return before we multiply, so this is clearly less efficient. To do something like this in constant stack space (a single stack frame), we use **Tail Recursion**.
## Tail Recursion

There are four key features in tail recursion:
- **Tail of a function**  
    The _tail_ is the chronologically last computation the function performs before returning. This has nothing to do with lists or `tl`.
    `n * fac (n - 1)`
    The multiplication is the tail, **not** the recursive call.
- **Tail call**  
    A function call that occurs _in the tail position_.  
    If the caller must “wait” to do more work after the call returns, it is **not** a tail call.
- **Tail recursive function**  
    A function is tail recursive ==if every recursive call is a tail call==.
- **Stack usage**  
    A tail recursive function **can** run in constant stack space (via tail-call optimization).  
    Non-tail recursion **cannot**.

Why `fac` is _not_ tail recursive
```OCaml
let rec fac n =
	if n = 0 then 1
	else n * fac (n - 1)
```
- `fac (n - 1)` returns a value
- Then we multiply by `n`
- That means work happens _after_ the recursive call

*A tail recursive version*
```OCaml
let facing f n =
	if n = 0 then f
	else facing (n * f) (n - 1) ;; (* tail call *)
let fac n =
	facing 1 n ;; (* tail call *)
```
- The recursive call is the final action
- Nothing happens after it each call returns
- Each call hands off _everything it needs_ to the next call, then exits.
Every new facing pops the other, so we only keep one facing frame on the stack at any time -- constant stack space.

### Exam Question
> "This might show up on an exam: You may be asked to see if a function is tail recursive, and if so expalin why. If not, you may be asked to rewrite it in tail recursive form, which may or may not be possible"

Checking if a function is tail recursive:
```OCaml
let rec foo x =
  if x = 0 then 1
  else x * foo (x - 1)
```
1. Locate all recursive calls
    - Here: `foo (x - 1)`
2. Check what happens after the call returns
    - After `foo (x - 1)` returns, we multiply it by `x`    
    - There’s still work to do → **not tail recursive**
3. Determine tail position
    - Only the _final action_ of the function is in tail position
    - Anything like `n * foo (n-1)`, `1 + f(...)`, `List.map f xs @ ...` is **not** tail position

Writing a function in recursive form:

*Non-tail recursive form*
```Ocaml
let rec fac n =
  if n = 0 then 1
  else n * fac (n - 1)
```

Add accumulator:
*Start with acc = 1, multiply before recursion*
```Ocaml
let rec fac_aux acc n =
  if n = 0 then acc
  else fac_aux (acc * n) (n - 1)
```

Move computation *into* recursive call:
```Ocaml
let rec fac_aux acc n =
  if n = 0 then acc
  else fac_aux (acc * n) (n - 1)
```

Add a wrapper:
```Ocaml
let fac n = fac_aux 1 n
```



