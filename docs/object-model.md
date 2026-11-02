# 6. Object-model conventions and concepts

## Member scoping and the `stranger` concept

Inside a class definition body:
- Identifiers prefixed with a leading dot (`.`) are **instance members** (attributes or instance methods).
- Identifiers prefixed with `cls.` are **class members** (class methods).
- Any identifier declared or referenced without a dot or `cls.` prefix is an external entity, termed a **stranger**. Strangers do not belong to the instance or class state.

## Constructor and attributes: `#from(...)`

The `#from(...)` directive defines the constructor parameters and automatically initializes instance attributes of the same names.

- An asterisk prefix (`*`) marks a **required parameter**:
  ```onyx
  #from(* data: real, grad: real = 0.0, label: string = "")
  ```
- Every parameter not marked with `*` **must** provide a default value. Omitting a default value for an unstarred parameter is a compilation error.
- Parameters declared in `#from(...)` become instance attributes accessible via `.attribute_name`.

## Methods: instance and class methods

The same dot convention distinguishes instance members from class members:

- **Instance methods** are declared with a leading dot (`.method_name(...) -> ReturnType:`).
  - They are invoked on a receiver: `receiver.method(arguments)`.
  - Inside the method body, instance attributes and methods are accessed via `.member`.
  - `self` refers to the receiver instance only when it is explicitly passed as a value to another function or constructor.
- **Class methods** are declared with the `cls.` prefix (`cls.method_name(...) -> ReturnType:`).
  - They are invoked directly on the class: `Class.method(arguments)`.
  - Inside a class method body, class members are reached through `cls.`. Class methods commonly serve as constructor helpers or factory functions.

Every method signature requires an explicit return type (`-> ReturnType:`), and every control-flow path must terminate with `=> expression`.

## Inheritance: `#parents(...)`

The `#parents(...)` directive declares antecedent components from which the class derives:

```onyx
#parents(BaseClass)
```

A derived class inherits methods, attributes, and characteristics defined in its parents.

**To decide:** multiple inheritance, method overriding, super calls, and composition semantics.

## String representation: `#format`

The `#format` directive defines the canonical textual representation of an instance (used by `print` and string interpolation):

```onyx
#format = "Value({.data}, grad={.grad}, label={.label})"
```

Instance attributes are referenced inside format strings using the leading dot notation (`{.attribute}`).
