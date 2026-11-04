# 10. Conformance examples to add

For each important rule, add a short program whose result or error is exact.
These examples become the first interpreter/compiler test suite.

```onyx
(< Numeric promotion. Expected type: real; expected value: 2.5. >)
value = 2 + 0.5

(< Static typing. Expected: compilation error because count is int. >)
count: int = 7
count = "seven"

(< 1. Scope: indentation delimits scopes.
   Expected output: inner = 2, outer = 1. >)
x: int = 1
if true:
    x: int = 2
    print("inner = {x}")
print("outer = {x}")

(< 2. Consumption across scopes: a parent is invalid after a block if any branch can transfer it.
   Expected: compilation error at print(a): a may have been transferred. >)
a: int = 7
if condition:
    b: int = ~a
print(a)

(< 3. Redeclaration: creates a new binding dominating the older one in current scope.
   Expected output: 2. >)
value: int = 1
value: int = 2
print(value)

(< 4. Warning: unread and untransferred dominated binding emits a warning, never an error.
   Expected warning: the first unused binding was never read or transferred. >)
unused: int = 1
unused: int = 2

(< 5. Reads: calculations, comparisons, calls, print, and indexing never transfer ownership.
   Expected output: 7, 8. >)
source: int = 7
next: int = source + 1
print(source)
print(next)

(< 6. Consumption: a variable is consumed when its value changes owner.
   Expected: compilation error at print(source): source was consumed by transfer to target. >)
source: int = 7
target: int = ~source
print(target) (< 7 >)
print(source)

(< 7. Reassignment: an assignment targets an invalid/consumed binding and resurrects it.
   Expected output: 9. >)
source: int = 7
target: int = ~source
source = 9
print(source)

(< 8. Transfer symbol: ~ marks explicit transfer; ordinary assignment does not transfer.
   Expected output: 7, 7. >)
first: int = 7
second: int = first (< read only: first remains valid >)
print(first)
print(second)

(< Persistent variable (!p): ~ does not consume a persistent binding.
   Expected output: 7, 7. >)
!p persistent_val: int = 7
transferred: int = ~persistent_val
print(transferred) (< 7 >)
print(persistent_val) (< 7 >)

(< Repeated transfer: compilation error because source is transferred twice. >)
source: int = 7
target = ~source + ~source

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
