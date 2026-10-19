# 5. Language constructs

## Lambdas

```onyx
lam_func = lambda (n: int) = n + 1
result = lam_func(2)
print("And the result is: {result}") (< It should be 3 >)
print("And the result is: {lam_func(8)}")

(< Zero-parameter lambda: parentheses are always required, never omitted. >)
no_op = lambda () = none
```

There is only one way to use this: only as shown in the example. `=`
separates the parameter list from the body -- no return type is written
after the parentheses, and the parentheses are never omitted, even with
zero parameters.

## Named functions

A named function has a parameter list, a required return type annotation, and
an indented body. The `->` between the parameter list and the return type is
part of the function signature. `=> expression` returns immediately from the
current function.

```onyx
add(a: int, b: int) -> int:
    => a + b

absolute(n: int) -> int:
    if n < 0:
        => -n
    => n

display(text: string) -> none:
    print(text)
    => none
```

`=>` is a return construct, not a lambda syntax. A function return type is
checked against the type of every returned expression.

There is no implicit return. Every path through a body must return
explicitly, so a function or method with return type `-> none` ends with
`=> none` rather than falling off its last statement.

An `arr` parameter grants the function mutable access to the passed array
storage. Mutations therefore affect the caller's array; no `mut` marker is
required in the first version.

```onyx
selection_sort(values: arr[int] <n>) -> arr[int] <n>:
    values[0] = 0
    => values
```

After `selection_sort(numbers)`, `numbers` contains the modified values. The
precise model for assignment, aliasing, copies, and views remains open.

`::` is reserved for a future `process` construct and has no meaning in the
current language.

**To decide:** recursion, function overloading, named-function scope, and
closure capture.

## Classes

```onyx
(< Simulating Value class from micrograd: >)

class Value:
    #from(* data: real, children: tuple = (), op: string = "", label: string = "") (<They all become attributes. `data` is the only required one -- the others must have a default value, or it's a compile/interpret error.>)
    .grad: real = 0.0 (< All variables preceded of a dot belong to the instance. >)
    .backward: none = lambda () = none
    #format = "Value({.data})" (<Formatting function for `Value` instance.>)

    (< An instance method: the leading dot marks it as belonging to each
       instance. Inside its body, the receiver's members keep the same dot
       marker; `self` names the receiver only when it is passed as a
       value. >)
    .add(other: Value) -> Value:
        => Value(.data + other.data, (self, other), "+", "add")

    (< A class method: the leading `cls.` marks it as belonging to the
       class itself. Class members are reached through `cls.` the same
       way instance members are reached through the dot. >)
    cls.constant(value: real) -> Value:
        => Value(value, (), "const", "constant")

weight: Value = Value(2.0, (), "input", "w")
input: Value = Value(-4.0, (), "input", "x")
product: Value = weight.mul(input)    (< instance member access >)
unit: Value = Value.constant(1.0)     (< class member access >)
```

A member preceded by a dot is an instance member: an attribute or a
method. An instance method is called on a receiver with
`receiver.method(arguments)`; the declared parameters are the remaining
arguments. Inside a method body, the receiver's members use the same dot
marker as in the class body -- `.data`, `.grad`, or a call such as
`.mul(other)` -- and `self` names the receiver when it is needed as a
value. A member preceded by `cls.` is a class method; it is called on
the class itself: `Class.method(arguments)`, and its body reaches class
members through `cls.` in the same way.

Method signatures and bodies follow the rules of named functions: the
return type after `->` is required, and `=>` returns from the method.

**To decide:** inheritance and composition, method overriding, and
whether `self` and `cls` may be rebound.

## Output and string formatting

```onyx
var: string = "programming language."
print("How to use a {var}")
print('How to use   {var}')
```

## Pattern matching

`match` selects a branch according to the value being tested.

```onyx
value = 9

match value:
    case 1:
        print("Monday")
    case 2:
        print("Tuesday")
    case 3:
        print("Wednesday")
    case 4:
        print("Thursday")
    case 5:
        print("Friday")
    case 6:
        print("Saturday")
    case 7:
        print("Sunday")
    default:
        print("Value error encountered")
```

## Error handling: `try` / `fallback`

The `try` block is evaluated and `fallback` is executed if it cannot complete
because of an error. Error propagation, rollback behaviour, and access to the
original error remain to be defined.

```onyx
try:
    this = Value(2.6)
fallback:
    print("Error: Failed to initialize this.")
```

## Conditional execution

`if`, `elif`, and `else` select a block from boolean conditions.

```onyx
real_var = 0.366565649486516846584684

if real_var > 1:
    print("{real_var} is greater than one.")
elif real_var < 1:
    print("{real_var} is less than one")
else:
    print("{real_var} is equal to 1")
```

## `while` loops

A `while` loop repeats its body while its condition evaluates to `bool`.

```onyx
index: int = 0
while index < 3:
    print("index = {index}")
    index = index + 1
```

`=> expression` inside a loop returns from the enclosing function immediately.
It consequently exits every active loop before control leaves the function.

`:break` exits only the innermost active loop. When a `while` is used as an
expression, `:break value` exits that loop and makes `value` the result of the
`while` expression. A `while` that ends through `:break` without a value, or
whose condition becomes `false`, evaluates to `none`.

```onyx
current: int = 5

position: int = while true:
    if current >= 9:
        :break current
    current += 1
```

`continue` is reserved for future design.

## Blocks, scope, and lifetime

Every indented block creates a child lexical scope. A child scope can read and
modify variables declared in any enclosing parent scope. A parent scope cannot
access variables declared only in one of its child scopes.

```onyx
total: int = 0

while total < 3:
    increment: int = 1

    if total >= 0:
        total += increment

print(total)     (< Valid: 3 >)
print(increment) (< Error: increment is out of scope >)
```

Scope determines name visibility. Lifetime determines how long a value must
remain available in memory; its exact implementation is part of the memory
model and may be managed by the compiler. A value can outlive its declaring
block only through an explicit mechanism such as a function return or a closure
capture.

**To decide:** shadowing, closure capture rules, and the exact lifetime and
aliasing model for values that escape a block.
