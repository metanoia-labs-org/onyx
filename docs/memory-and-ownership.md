# Memory and ownership

## Mutability, type stability, and consumption

Onyx distinguishes a mutable binding, a mutable value, and a consumed value.
These are related, but they are not the same concept.

- **Bindings are mutable by default.** A name may be rebound to another value.
- **A binding's type is stable.** Once inferred or annotated, it cannot be
  rebound to an incompatible type.
- **Arrays are mutable in place.** A compatible value may be assigned to an
  indexed array cell.
- **Linear transformations consume their operand** according to the rules in
  this document. Consumption makes a name unavailable; it does not make
  bindings immutable in general.
- **`!p` means persistent, not immutable.** It prevents consumption by a
  transformation, while reassignment and in-place mutation remain permitted.

```onyx
count: int = 7
count = 8                    (< valid: mutable binding, same type >)

values: arr[int] <3> = [1, 2, 3]
values[0] = 9                (< valid: in-place mutation >)

!p persistent: arr[int] <3> = [1, 2, 3]
persistent[0] = 9            (< valid: !p is not immutability >)
```

## Consumption, persistence, and sharing

A variable in onyx is **linear** by default: when its value is used to
produce a new value of the same kind, the variable is **consumed** — it
becomes invalid, and the new value takes its place. This makes memory
ownership explicit without requiring pointers or manual allocation.

## What consumes a variable

A variable is consumed only when it appears as a direct operand in a
**transformation** — an operation that produces a new value of the same kind
(an `arr` producing an `arr`, a class producing a class, and so on).

The following operations **consume** their `arr` operand:

- direct assignment: `b = a`
- array-returning functions: `b = sort(a)`, `b = map(f, a)`
- array arithmetic producing a new array: `b = a + c`

The following operations **do not consume** their operand:

- reads and projections: `print(a)`, `a[0]`, `a.length`, `a.shape`
- scalar-producing reductions: `sum(a)`, `first(a)`, `product(a)`
- scalar comparisons and boolean tests

In every non-consuming case, the variable remains valid after the
operation.

## Repeated use in one consuming expression

A linear variable cannot appear twice as a consuming operand in the same
expression. Each such operand would require ownership of the same value, which
would make the evolution ambiguous.

```onyx
a: arr[int] <3> = [1, 2, 3]
b = a + a        (< compilation error: a is consumed twice >)
```

```onyx
!p a: arr[int] <3> = [1, 2, 3]
b = a + a        (< valid: a remains available >)
```

`copy(a)` is the general explicit solution. It preserves `a` for the first
operand, while the second operand consumes it:

```onyx
b = copy(a) + a
```

This requires a full copy followed by the array operation: both are `O(n)`, so
the expression performs two array-sized passes and requires one array-sized
temporary value. A specialized transformation can be more efficient:

```onyx
b = double(a)
```

`double` consumes `a` once and can evolve its storage in place, requiring one
`O(n)` pass and `O(1)` additional storage. Such specialized operations are the
preferred performance path when their semantics match the intended work.

```onyx
a: arr[int] <3> = [1, 2, 3]

b = a              (< consumes a: a is no longer valid >)
print(a)           (< error: a was consumed by the previous line >)
```

```onyx
a: arr[int] <3> = [1, 2, 3]

n: int = sum(a)    (< does not consume a >)
m: int = a[0]      (< does not consume a >)
print(a)           (< valid: a = [1, 2, 3] >)
```

## Consumption across scopes

A name declared in a child scope ceases to exist when that scope closes. A
variable of an enclosing scope that is consumed inside a child scope stays
consumed after the block, if any path through the block may consume it.

```onyx
a: arr[int] <3> = [1, 2, 3]

if condition:
    b = a        (< consumes a; b exists only inside this block >)
                 (< end of block: b ceases to exist >)

print(a)         (< error: a may have been consumed by the block >)
```

Validity is decided at compile time, from the text of the program alone. It
never depends on the value of a condition.

## Persistent variables (`!p`)

A variable declared with the `!p` prefix is **persistent**. Persistent
variables are never consumed. When a transformation would consume a
persistent variable, the language operates on a copy instead; the original
variable remains valid and unchanged.

```onyx
!p a: arr[int] <3> = [1, 2, 3]

b = a              (< a is not consumed; b shares a's value >)
c = sort(a)        (< a is not consumed; c is a sorted copy >)

print(a)           (< [1, 2, 3] >)
print(c)           (< [1, 2, 3] >)
```

Persistent variables share their storage with every variable they are
assigned to, until a mutation forces a copy. This sharing is invisible to
the programmer: independent mutations always produce independent results.

```onyx
!p a: arr[int] <3> = [1, 2, 3]

b = a              (< b shares a's storage >)
c = a              (< c shares a's storage >)
d = a              (< d shares a's storage >)

d[0] = 77          (< a copy is created for d; a, b, c unchanged >)

print(a)           (< [1, 2, 3] >)
print(b)           (< [1, 2, 3] >)
print(c)           (< [1, 2, 3] >)
print(d)           (< [77, 2, 3] >)
```

**Note.** `!p` does not affect mutability. A persistent variable can be
reassigned and mutated exactly like a linear one. `!p` only controls whether
the variable is consumed by transformations.

## Explicit copy

For a linear variable that must survive one specific operation without being
declared persistent for its entire lifetime, `copy(expression)` produces an
independent value immediately.

```onyx
a: arr[int] <3> = [1, 2, 3]

b = copy(a)        (< a survives this line; b is independent >)
c = a              (< consumes a here >)

b[0] = 77          (< b is independent of c >)

print(b)           (< [77, 2, 3] >)
print(c)           (< [1, 2, 3] >)
```

`copy` is not required on a variable already declared `!p`, because the
language already guarantees that the original survives. Applying `copy` to
such a variable is permitted, but it forces an immediate physical copy
instead of a shared representation.

## Relation to scope and lifetime

Consumption affects **which names remain valid**, not the lifetime of the
storage itself. When a value is consumed, its storage may be transferred to
the consuming variable, or released when it is no longer referenced. The
compiler chooses the cheapest strategy that preserves the visible semantics.

Scope and lifetime are therefore distinct concerns:

- **Scope** determines name visibility (section *Blocks, scope, and lifetime*).
- **Consumption** determines whether a name remains valid after a transformation.
- **Lifetime** determines how long the underlying storage must remain alive.

A value can outlive its declaring scope only through an explicit mechanism,
such as a function return or a closure capture.

## Summary

| Operation | Consumes the operand? |
| --- | --- |
| `b = a` | yes, unless `a` is `!p` |
| `b = sort(a)`, `b = map(f, a)` | yes, unless `a` is `!p` |
| `b = a + a` (array) | compilation error for a linear `a`; valid for `!p a` |
| `print(a)`, `a[0]`, `a.length`, `a.shape` | no |
| `sum(a)`, `first(a)`, `product(a)` | no |
| `b = copy(a)` | no |
| numeric scalars (`int`, `real`, `complex`, `quaternion`, `bool`) | never consumed; always copied |
| `string` | never consumed; compared and concatenated by value |

**To decide:** the exact set of consuming operations, the behavior of a `while`
whose body consumes a variable of an enclosing scope (where a second iteration
would find it already consumed), and the interaction with closures and function
returns.
