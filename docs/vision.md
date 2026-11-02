# 1. Vision and scope

`onyx` is an initiative for modern, accurate programming, combining performance
with a compact syntax. It is a research language intended to serve several
purposes in computing, with native support for numerical programming and
multidimensional tensors.

This document is organised from the smallest concepts (syntax, values and
expressions) to the most advanced ones (tensors, objects and implementation).
Rules in the final **Open Design Questions** section are not yet part of the
stable language specification.

## Design principles

- Prefer predictable rules over many special cases.
- Keep type annotations optional when inference is unambiguous.
- Use static typing and compile-time verification to make memory, ownership,
  and performance guarantees dependable.
- Make numerical conversion and tensor-shape rules explicit.
- Specify errors and edge cases before relying on them in an implementation.
