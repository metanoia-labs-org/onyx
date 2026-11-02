# 10. Conformance examples to add

For each important rule, add a short program whose result or error is exact.
These examples become the first interpreter/compiler test suite.

```onyx
(< Numeric promotion. Expected type: real; expected value: 2.5. >)
value = 2 + 0.5

(< Static typing. Expected: compilation error because count is int. >)
count: int = 7
count = "seven"

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
