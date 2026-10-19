# 8. Open design questions

| # | Topic | Status |
| -- | ----- | ------ |
| 1 | Expression semantics (operators, precedence, function calls, assignment, and evaluation order) | partially defined |
| 2 | Variable mutability (can variables and attributes be reassigned? const vs. mutable?) | partially defined |
| 3 | Scoping and closures (how `lambda` captures variables from its enclosing scope) | partially defined |
| 4 | Functions and closures (named functions, return behavior, recursion, and closure lifetime) | partially defined |
| 5 | Type system (type inference, explicit annotations, generic types, and compatibility rules) | undone |
| 6 | `template` keyword (syntax, declaration scope, instantiation, and interaction with generic types) | undone |
| 7 | Numeric promotion (conversion rules between `int`, `real`, `complex`, and `quaternion`, including precision and overflow) | partially defined |
| 8 | Error model (whether an error is a type, code/message pair, class, or another value) | undone |
| 9 | `try`/`fallback` semantics (error propagation, rollback behavior, and access to the original error) | undone |
| 10 | Class and object model (constructors, fields, methods, `self`, inheritance, and composition) | partially defined |
| 11 | `#from`, `#parents`, and `#format` semantics (whether they are keywords, directives, or metadata annotations) | undone |
| 12 | Member terminology (formal definition of “stranger” and the distinction between instance, class, and external variables) | undone |
| 13 | Tensor indexing and slicing (index syntax, bounds checking, negative indices, and slice semantics) | partially defined |
| 14 | Tensor operations (broadcasting, reshaping, reductions, concatenation, and elementwise versus matrix multiplication) | undone |
| 15 | Tensor shape system (compile-time versus runtime dimensions and shape mismatch behavior) | undone |
| 16 | Memory and execution model (layout, ownership, allocation, copying, views, and performance guarantees) | partially defined |
| 17 | Modules and interoperability (imports, packages, foreign-function interfaces, and external libraries) | undone |
| 18 | Diagnostics and tooling (compiler errors, warnings, formatting, testing, debugging, and documentation conventions) | undone |
| 19 | Implementation roadmap (lexer, parser, AST, type checker, interpreter/compiler, standard library, and test suite) | undone |
