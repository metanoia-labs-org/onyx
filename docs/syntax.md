# 2. Core syntax

## Blocks and comments

Blocks are introduced by `:` and delimited by indentation. Comments use `(<`
to open and `>)` to close; they can be single-line or multi-line.

```onyx
if active:
    print("Running")

(< This is a comment. >)
```

**To decide:** indentation policy (spaces/tabs, width) and whether empty blocks
are allowed.

## Variable declarations

Variables can be declared using inference or an explicit type annotation.

```onyx
variable_name = data
variable_name: data_type = data
```

Onyx is statically typed. When a declaration omits its annotation, the
compiler infers the type from the assigned value; it does not defer the type
decision to runtime.

```onyx
count = 7                 (< inferred as int >)
values = [1, 2, 3]        (< inferred as arr[int] <3> >)
limit: int = 10           (< explicit contract >)
```

Once inferred or annotated, a binding cannot later hold a value of an
incompatible type. An explicit annotation is especially useful for function
parameters, return types, public APIs, and values whose intended precision or
shape must be clear.

Bindings are mutable by default. A later assignment to an existing variable is
permitted:

```onyx
value = 1
value = 2
```

**To decide:** constants and declarations without a value.
