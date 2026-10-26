# Memory and ownership

## 7. Consumption, persistence, and sharing

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
- array arithmetic producing a new array: `b = a + a`

The following operations **do not consume** their operand:

- reads and projections: `print(a)`, `a[0]`, `a.length`, `a.shape`
- scalar-producing reductions: `sum(a)`, `first(a)`, `product(a)`
- scalar comparisons and boolean tests

In every non-consuming case, the variable remains valid after the
operation.

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

## Persistent variables (`!i`)

A variable declared with the `!i` prefix is **persistent**. Persistent
variables are never consumed. When a transformation would consume a
persistent variable, the language operates on a copy instead; the original
variable remains valid and unchanged.

```onyx
!i a: arr[int] <3> = [1, 2, 3]

b = a              (< a is not consumed; b shares a's value >)
c = sort(a)        (< a is not consumed; c is a sorted copy >)

print(a)           (< [1, 2, 3] >)
print(c)           (< [1, 2, 3] >)
```

Persistent variables share their storage with every variable they are
assigned to, until a mutation forces a copy. This sharing is invisible to
the programmer: independent mutations always produce independent results.

```onyx
!i a: arr[int] <3> = [1, 2, 3]

b = a              (< b shares a's storage >)
c = a              (< c shares a's storage >)
d = a              (< d shares a's storage >)

d[0] = 77          (< a copy is created for d; a, b, c unchanged >)

print(a)           (< [1, 2, 3] >)
print(b)           (< [1, 2, 3] >)
print(c)           (< [1, 2, 3] >)
print(d)           (< [77, 2, 3] >)
```

**Note.** `!i` does not affect mutability. A persistent variable can be
reassigned and mutated exactly like a linear one. `!i` only controls whether
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

`copy` is not required on a variable already declared `!i`, because the
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
| `b = a` | yes, unless `a` is `!i` |
| `b = sort(a)`, `b = map(f, a)` | yes, unless `a` is `!i` |
| `b = a + a` (array) | yes, unless `a` is `!i` |
| `print(a)`, `a[0]`, `a.length`, `a.shape` | no |
| `sum(a)`, `first(a)`, `product(a)` | no |
| `b = copy(a)` | no |
| numeric scalars (`int`, `real`, `complex`, `quaternion`, `bool`) | never consumed; always copied |
| `string` | never consumed; compared and concatenated by value |

**To decide:** the exact set of consuming operations, the behavior when a
variable is used twice in the same expression (e.g. `b = a + a`), the behavior
of a `while` whose body consumes a variable of an enclosing scope (where a
second iteration would find it already consumed), the interaction with closures
and function returns, and the final name of the `!i` prefix.
