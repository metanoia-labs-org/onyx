# 3. Values and types

## Basic data types

Onyx provides the following fundamental data types:

- `string`
- `int` (including sized variants `int32`, `int64`, `int128`, `int256`)
- `real` (including sized variants `r32`, `r64`, `r128`, `r256`)
- `complex`
- `quaternion`
- `bool`
- `arr` (multidimensional tensors)
- `none`

There is seamless interoperability between numerical types (`int`, `real`, `complex`, and `quaternion`), enabling binary arithmetic operators to combine them directly.

## Static typing and inference

Onyx is a statically typed language with strong type inference. Every value and
binding has a type known to the compiler before execution. Type errors are
therefore compilation errors, rather than failures that appear only when a
particular runtime path is reached.

```onyx
count: int = 7
count = "seven" (< compilation error: string is not compatible with int >)
```

Annotations are optional when a type can be inferred unambiguously:

```onyx
count = 7                    (< inferred as int >)
ratio = 0.5                  (< inferred as real >)
values = [1, 2, 3]           (< inferred as arr[int] <3> >)
```

Static typing supports Onyx's memory model: the compiler can verify
consumption, ownership transfers, tensor shapes where known, and incompatible
transformations before the program runs. It also allows optimizations without
requiring runtime type checks in performance-sensitive code.

## Numeric promotion

Numeric operations follow a single widening rule rather than a long per-operator list.
The hierarchy is:

`int -> real -> complex -> quaternion`

The result of a binary numeric operation is the widest numeric type present in the operands.
This means the following are the canonical cases:

- `int + int = int`
- `int + real = real`
- `real + real = real`
- `int + complex = complex`
- `real + complex = complex`
- `complex + complex = complex`
- `int + quaternion = quaternion`
- `real + quaternion = quaternion`
- `complex + quaternion = quaternion`
- `quaternion + quaternion = quaternion`

The same rule applies to subtraction, multiplication, and division among
numeric values. Exponentiation involving quaternions remains to be specified.
This gives a compact and predictable rule set without requiring every pairwise combination to be listed explicitly.

Examples:

- `2 + 2 = 4`
- `1 + 2.856295 = 3.856295`
- `2.52 + 2.48 = 5.0`
- `(2 + 52i) + (2 + 48i) = 4 + 100i`
- `1.79809804 + (2 + 8i) = 3.79809804 + 8i`
- `2 - 2 = 0`
- `1 - 2.856295 = -1.856295`
- `2.52 - 2.48 = 0.04`
- `(2 + 52i) - (2 + 48i) = 4i`
- `1.79809804 - (2 + 8i) = -0.20190196 - 8i`
- `2 * 2 = 4`
- `1 * 2.856295 = 2.856295`
- `2.52 * 2.48 = 6.2496`
- `(2 + 52i) * (2 + 48i) = -2492 + 200i`
- `1.79809804 * (2 + 8i) = 3.59619608 + 14.38478432i`
- `2 / 4 = 0.5`

This rule keeps the language consistent with the usual arithmetic hierarchy while separating numeric semantics from non-numeric operations such as string concatenation and tensor behavior.

The exact treatment of precision, overflow, integer division, and division by zero remains a design question to be resolved in the numeric model.

## Arithmetic operators

- `+` adds numeric values according to numeric promotion. `string + string`
  concatenates strings.
- `-` and `*` operate on numeric values according to numeric promotion.
- `/` operates on numeric values according to numeric promotion. Division of two
  integers yields a `real` result.
- `//` is integer division. It accepts integers only and returns an `int`.
  Its rounding rule for negative operands remains part of the numeric model.
- `%` is the integer remainder operator. It accepts integers only. For a positive
  divisor `b`, `a % b` is the unique remainder `r` such that
  `a = b * q + r` and `0 <= r < b`. Division by zero is an error.
- `^` is numeric exponentiation. It has a higher precedence than `*` and `/` and
  is right-associative: `2 ^ 3 ^ 2` means `2 ^ (3 ^ 2)`. Quaternion powers are
  defined in the **Quaternions** section.
- Unary `+value` and `-value` are supported for numeric values.
- `ln()` and `exp()` accept `int`, `real`, `complex`, and `quaternion` values.
  Quaternion semantics are defined in the **Quaternions** section.
- `abs()` accepts numeric values. It returns the modulus of a complex number and
  the norm of a quaternion, in both cases as a `real`.

The precise handling of non-integer powers, domains, precision, overflow, and
exceptional floating-point values remains part of the numeric model.

## Comparison operators

Onyx provides `==`, `!=`, `<`, `<=`, `>`, and `>=`.

- Numeric comparisons apply numeric promotion, so `2 == 2.0` is `true`.
- Strings are compared by their contents.
- `none == none` is `true`.
- Ordering comparisons (`<`, `<=`, `>`, `>=`) are invalid for `complex` and
  `quaternion`, because these types have no natural total order.
- Tensor comparison semantics are intentionally deferred.

## Boolean operators

Onyx uses keyword boolean operators:

```onyx
not active
is_admin and has_access
is_guest or is_admin
```

`and` and `or` use short-circuit evaluation. Conditions in `if`, `elif`, and
other control-flow constructs must evaluate to `bool`; values such as `0`, an
empty string, and an array are not implicitly converted to `bool`.

## Operator precedence

The currently defined precedence, from highest to lowest, is:

1. Function calls and indexing (indexing syntax remains open).
2. `^` (right-associative).
3. Unary `+` and unary `-`.
4. `*`, `/`, and `%`.
5. `+` and `-`.
6. `<`, `<=`, `>`, and `>=`.
7. `==` and `!=`.
8. `not`.
9. `and`.
10. `or`.

Arithmetic operators at the same precedence level are evaluated from left to
right, except `^`.

Array operations, including elementwise operations, matrix products, and
broadcasting, remain to be specified separately.

## Quaternions

A `quaternion` has four real components and is written mathematically as
`a + bi + cj + dk`. It extends `complex` through the subspace `a + bi`.

Quaternions are constructed with the canonical component order `(real, i, j, k)`:

```onyx
q: quaternion = quaternion(1, 2, 3, 4) (< 1 + 2i + 3j + 4k >)
```

The units follow the standard multiplication rules:

```text
i² = j² = k² = -1
ij = k, jk = i, ki = j
ji = -k, kj = -i, ik = -j
```

Quaternion multiplication is associative but not commutative. Therefore,
`a * b` and `b * a` must not be assumed equal when either operand is a
quaternion. The language fixes `i` as the complex unit embedded in a
quaternion, making promotion from `complex` to `quaternion` unambiguous.

`abs(q)` returns the real quaternion norm. Division by the zero quaternion is
an error.

`exp(q)` is defined for every quaternion. `ln(q)` is defined for every non-zero
quaternion and uses a deterministic principal branch. For a negative real value
represented as a quaternion, whose imaginary axis is otherwise ambiguous, the
principal logarithm uses the `i` axis:

```onyx
ln(quaternion(-2, 0, 0, 0)) (< ln(2) + πi >)
```

Quaternion powers accept integer and real exponents:

- `q ^ n`, where `n` is an `int`, uses integer powers; `q ^ 0` is the
  multiplicative identity `quaternion(1, 0, 0, 0)`.
- `q ^ r`, where `r` is a `real`, is defined as `exp(r * ln(q))` using the
  principal logarithm.
- A quaternion exponent is not supported in the first version.

All mathematical operations defined here follow standard mathematical semantics.
`token` and `map` are deferred until their syntax and semantics are settled.
