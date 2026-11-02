# Onyx contributor guide

## Project purpose

Onyx is an experimental statically typed systems language. Its central design
goal is predictable performance and memory ownership without exposing pointers
or manual allocation to application code.

The language is a specification in progress. Preserve decisions marked as
defined; do not silently invent semantics for items listed as open questions.

## Documentation map

- `docs/vision.md`: language purpose and design principles.
- `docs/syntax.md`: declarations, annotations, blocks, and surface syntax.
- `docs/values-and-types.md`: scalar types, promotion, operators, and type
  inference.
- `docs/tensors.md`: `arr` declarations, indexing, shape, and tensor rules.
- `docs/language-constructs.md`: functions, classes, control flow, and scope.
- `docs/memory-and-ownership.md`: mutability, linear consumption, `!p`, and
  explicit copying.
- `docs/open-questions.md`: unresolved design work and its status.
- `docs/conformance-examples.md`: small examples intended to become tests.
- `snippets/`: larger executable-style examples used to pressure-test syntax.

When a decision changes, update the authoritative topic file first, then any
affected snippet and open-question status.

## Non-negotiable language decisions

- Onyx is **statically typed** with strong inference. Omitted annotations are
  inferred at compile time; bindings never change to an incompatible type.
- Bindings are mutable by default. Arrays (`arr`) are mutable in place.
- Variables holding linear values evolve through consuming transformations.
  A consumed name is unavailable after the transformation.
- `!p` means **persistent**, not immutable and not global. It prevents
  consumption by transformations, but does not prevent reassignment or in-place
  mutation.
- `copy(expression)` creates an independent value explicitly.
- A linear value cannot occur twice as a consuming operand in one expression:
  `a + a` is a compilation error for linear `a`.
- Reads, projections, scalar reductions, and boolean tests do not consume their
  operands.
- Indentation delimits lexical scopes. A child scope can access parent names;
  a parent cannot access child-only names.
- `=> value` returns from the enclosing function. `:break` leaves only the
  innermost loop; `:break value` supplies the result of a `while` expression.

## Writing Onyx examples

- Use `(< ... >)` for comments.
- Use `name: Type = value` for explicit annotations; rely on inference only
  when the example is intentionally demonstrating it.
- Named functions use `name(parameters) -> ReturnType:` and terminate every
  path with `=> value`.
- Keep examples aligned with settled rules. If an example deliberately explores
  an open feature, label it clearly as a draft and name the assumptions.
- Do not use `!i`; the persistent marker is `!p`.
- Avoid introducing syntax for `map`, `token`, `template`, slices, generics, or
  processes unless the related specification has been settled.

## Editing and verification

- Keep documentation in English; discuss design in French when appropriate.
- Prefer concise normative wording: say what is valid, invalid, or still open.
- Run `git diff --check` after edits.
- Do not commit or push unless explicitly requested.
