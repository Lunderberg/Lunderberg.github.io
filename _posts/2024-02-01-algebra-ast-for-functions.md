---
layout: post
title: The Road to Haskell is Paved with Good Intentions
tags: [math]
---

I'm putting together a rudimentary parser for a hobby project, and ran
into some knock-on effects resulting from what I had initially
considered to be internal implementation details.  The parser produces
a abstract syntax tree, which is to be used for algebraic
manipulations.  To make these easy to express, I want to support
pattern-matching in the AST.

1. Pattern matching can be applied to the syntax tree.
2. Pattern matching should not require dynamic allocations.

On success, the pattern matching returns a view into the AST, where
any referenced AST node `ChildNode` is replaced with `View<'view,
ChildNode>`.  If there is a list of child nodes, `Vec<Node>` in the
syntax tree would require generating a `Vec<View<'view, Node>>` when
producing a pattern-matched result.  While though a reference to a
`&'view Node` can be converted to `View<'view, Node>` without dynamic
allocations, converting from `&'view Vec<Node>` into
`Vec<View<'view,Node>>` would require dynamic allocation.  In some
languages, this could be resolved using partial specialization, to
handle `Vec<_>` as a special case, but the current implementation is
in Rust, which doesn't support partial specialzation.  Therefore, no
use of `Vec<_>`.

3. `Vec<_>` may not be used in the syntax tree.

Now it comes to a problem for function calls.  Most languages allow
functions to accept more than one argument (citation needed), but this
becomes difficult to represent without a vector of parameters.  I
could represent more than one parameter using a tuple of parameters,
but that would require representing tuples of arbitrary size, which
has the same problem.  Instead, I need to represent multiple
parameters using currying.

4. Functions with more than one parameter are represented by
   currying.  `|x, y| { x + y }` is equivalent to `|x| { |y| {
   x+y } }`, and `func(x,y)` is equivalent to `func(x)(y)`.
   
This works for adding more and more parameters to the function, with
each new parameter introducing another nested `|param| { ... }` to the
function defintion, and another `(arg)` when calling it.

The weird part comes when handling zero-parameter functions.  For each
parameter I remove from the function, I unwrap `|param| { body }`.
So, if I have a one-parameter function `let func = |x| { 2*3 }`,
called with `func(42)`, I could remove the unused parameter and have
`let func = 2*3`, called with `func`.  That is, there is no way to
distinguish between a zero-parameter function and an expression.

5. A zero-parameter function is the same as an expression.

Okay, I wasn't expecting that.  This works well for functions that
return a constant; `|| { 42 }` must always return `42` each time that
it is called.  This doesn't work well for functions that return a
different value each time they're called.  If I can't distinguish
between `|| { current_system_time }` and `current_system_time`, then I
can't determine when the value should be read out.  This is true for
any side effect, since the effect of a side effect can depend on when
the function gets evaluated.

Since I don't have any way to represent the point at which
zero-parameter functions are evaluated, I need to forbid side effects
altogether in zero-parameter functions,

6. Zero-parameter functions must be pure.

At first, this looks like I can breathe a sigh of relief: 
While zero-parameter functions must be pure, it looks like functions
with parameters could still have side effects.  However, 

A one-parameter function `|param| { body }` has an arbitrary
expression as its body.  Any expression is equivalent to a
zero-parameter function, so `|param| { body }` is equivalent to
`|param| { || { body } }`.  Since a zero-parameter function must not
have side effects, and `body` is equivalent to `|| { body }`, the
function body must not have side effects.

7. All functions must be pure.

At this point, the only way to express side effects is by either
returning a new value for use in the calling scope (e.g. `let x =
func(x)`) or by returning something that represents a side effect
(e.g. `let eval_for_current_time = || { get_current_time }`).  This
is the 

I had initially thought that I was just deciding how to internally
structure the AST, with no impact on the language design overall.
Each step from there makes sense to maintain internal consistency, but
ended up with a pretty foundational statement about the language
design.  Since the intended use for this AST is in algebraic
manipulations, I don't expect this to be a significant drawback, and
am looking forward to seeing what other side effects result from
design choices.
