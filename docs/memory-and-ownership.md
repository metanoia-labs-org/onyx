# Memory and ownership

## Mutability, type stability, and ownership

Onyx distinguishes a mutable binding, a mutable value, and ownership transfer.
These are related, but they are not the same concept.

- **Bindings are mutable by default.** A name may be rebound to another value.
- **A binding's type is stable.** Once inferred or annotated, it cannot be
  rebound to an incompatible type.
- **Arrays are mutable in place.** A compatible value may be assigned to an
  indexed array cell.
- **Ownership transfer is explicit.** A name becomes invalid (consumed) only
  when its value is transferred to a new owner with `~`.
- **`!p` means persistent, not immutable.** It preserves a binding through a
  requested transfer, while reassignment and in-place mutation remain
  permitted.

```onyx
count: int = 7
count = 8                    (< valid: mutable binding, same type >)

values: arr[int] <3> = [1, 2, 3]
values[0] = 9                (< valid: in-place mutation >)

!p persistent: arr[int] <3> = [1, 2, 3]
persistent[0] = 9            (< valid: !p is not immutability >)
```

## Reads and explicit transfer

Every ordinary use of a name is a read. Calculations, comparisons, function
calls, `print`, indexing, projections, and ordinary assignment never transfer
ownership. This rule applies to every type, including numeric scalars and
strings.

```onyx
a: int = 7
b: int = a + 1
print(a) (< valid: 7 >)
```

The `~` prefix is the only transfer marker. It changes the owner of a value
and invalidates the source name until that name is assigned again.

```onyx
a: int = 7
b: int = ~a
print(b) (< valid: 7 >)
print(a) (< compilation error: a was transferred to b >)
```

Transfers may appear wherever a value is accepted, including on the right side
of an assignment and in a function call:

```onyx
result = transform(~source)
```

An ordinary assignment such as `b = a` is non-transferring: `a` remains valid.
Its physical representation may use copying or sharing, but that choice never
changes the visible ownership rules.

## Reassignment after transfer

A transferred name is invalid, not permanently removed. An assignment targets
the visible binding and makes it valid again, provided the new value is
compatible with its established type.

```onyx
source: int = 7
target: int = ~source
source = 9
print(source) (< valid: 9 >)
```

## Consumption across scopes

A name declared in a child scope ceases to exist when that scope closes. A
variable of an enclosing scope that is transferred inside a child scope stays
consumed (invalid) after the block, if any path through the block may transfer it.

```onyx
a: int = 7

if condition:
    b: int = ~a  (< transfers a; b exists only inside this block >)
                 (< end of block: b ceases to exist >)

print(a)         (< compilation error: a may have been transferred by the block >)
```

Validity is decided at compile time, from the text of the program alone. It
never depends on the runtime value of a condition.

## Repeated transfer in one expression

A linear variable cannot be transferred twice in the same expression. Each
transfer requires exclusive ownership of the value.

```onyx
a: int = 7
b = ~a + ~a      (< compilation error: a is transferred twice >)
```

When multiple uses of a value are needed, ordinary reads do not transfer:

```onyx
a: int = 7
b = a + a        (< valid: reads do not transfer ownership >)
```

If one operand must be transferred while another is read, the read operand can
be read normally, or an explicit `copy` can be made:

```onyx
b = a + ~a       (< valid: first operand is read, second is transferred >)
```

## Persistent variables (`!p`)

A variable declared with the `!p` prefix is **persistent**. Persistent
variables cannot be consumed. When a transfer (`~`) is applied to a
persistent variable, the variable remains valid and available.

```onyx
!p a: arr[int] <3> = [1, 2, 3]

b: arr[int] <3> = ~a   (< a is not consumed; a remains valid >)
print(a)               (< [1, 2, 3] >)
print(b)               (< [1, 2, 3] >)
```

Persistent variables share their underlying storage until a mutation forces an
independent copy (copy-on-write). This sharing is invisible to the programmer:
mutations to one alias do not affect another.

```onyx
!p a: arr[int] <3> = [1, 2, 3]

b = a              (< b shares a's storage >)
c = a              (< c shares a's storage >)
d: arr[int] <3> = ~a (< d receives a copy/share; a remains valid >)

d[0] = 77          (< a copy is created for d; a, b, c unchanged >)

print(a)           (< [1, 2, 3] >)
print(b)           (< [1, 2, 3] >)
print(c)           (< [1, 2, 3] >)
print(d)           (< [77, 2, 3] >)
```

**Note.** `!p` does not affect mutability. A persistent variable can be
reassigned and mutated in place exactly like a linear one. `!p` only prevents
invalidation through transfer.

## Explicit copy

`copy(expression)` produces an independent value immediately:

```onyx
a: arr[int] <3> = [1, 2, 3]

b = copy(a)        (< a remains valid; b is an independent copy >)
c = ~a             (< transfers a here >)

b[0] = 77          (< b is independent of c >)

print(b)           (< [77, 2, 3] >)
print(c)           (< [1, 2, 3] >)
```

## Relation to scope and lifetime

Transfer affects **which names remain valid**, not the lifetime of the
storage itself. When a value is transferred, its storage may be moved to
the receiving variable, or released when it is no longer referenced. The
compiler chooses the cheapest strategy that preserves the visible semantics.

Scope, ownership, and lifetime are distinct concerns:

- **Scope** determines name visibility (section *Blocks, scope, and lifetime*).
- **Ownership transfer** determines whether a name remains valid after an expression.
- **Lifetime** determines how long the underlying storage must remain allocated.

A value can outlive its declaring scope only through an explicit mechanism,
such as a function return or a closure capture.

## Summary

All data types — including numeric scalars (`int`, `real`, `complex`, `quaternion`),
booleans, strings, arrays (`arr`), and class instances — follow the exact same
ownership and transfer rules. Scalars are not an exception.

| Operation | Transfers ownership (consumes operand)? |
| --- | --- |
| `b = ~a`, `call(~a)` | yes: `a` becomes invalid (unless `!p`) |
| `b = a` (ordinary assignment) | no: read; `a` remains valid |
| `print(a)`, `a[0]`, `a.length`, `a.shape` | no: read; `a` remains valid |
| arithmetic, reductions: `a + 1`, `sum(a)` | no: read; `a` remains valid |
| `b = copy(a)` | no: produces an independent copy; `a` remains valid |
| repeated transfer: `~a + ~a` | compilation error: `a` cannot be transferred twice |
| reassignment: `a = value` | no: restores (resurrects) an invalid binding |
| persistent variable: `~p` where `!p p` | no: `!p` bindings are immune to consumption |

**To decide:** the behavior of a `while` whose body transfers a variable of an
enclosing scope (where a second iteration would find it already transferred), and
the interaction with closures and function returns.
