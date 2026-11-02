# 10. Conformance examples to add

For each important rule, add a short program whose result or error is exact.
These examples become the first interpreter/compiler test suite.

```onyx
(< Numeric promotion. Expected type: real; expected value: 2.5. >)
value = 2 + 0.5

(< Static typing. Expected: compilation error because count is int. >)
count: int = 7
count = "seven"

(< Linear consumption. Expected: compilation error because a is consumed twice. >)
a: arr[int] <3> = [1, 2, 3]
b = a + a

(< Persistent variable. Expected: valid because a is persistent (!p). >)
!p a: arr[int] <3> = [1, 2, 3]
b = a + a

(< Explicit copy. Expected: valid because copy(a) creates an independent value. >)
a: arr[int] <3> = [1, 2, 3]
b = copy(a) + a

(< Non-consuming operations. Expected: valid, a remains available after reads and reductions. >)
a: arr[int] <3> = [1, 2, 3]
first: int = a[0]
total: int = sum(a)
size: int = a.length
print(a)

(< In-place array mutation. Expected: valid, values[0] is modified in place. >)
numbers: arr[int] <3> = [1, 2, 3]
numbers[0] = 9

(< Function return. Expected: compilation error because not every path terminates with =>. >)
invalid_fn(x: int) -> int:
    if x > 0:
        => x

(< Closure semantics. The expected result depends on the capture rule. >)
offset = 1
add_offset = lambda (n: int) = n + offset

(< Tensor indexing and layout must be specified before this has a result. >)
matrix: arr[int] <2, 2> = [1, 2, 3, 4]

(< Error model and fallback semantics must be specified. >)
try:
    value = 1 / 0
fallback:
    value = none
```
