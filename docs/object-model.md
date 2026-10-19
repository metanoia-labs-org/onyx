# 6. Object-model conventions and concepts

- The `stranger` concept

    In a class/data structure definition,
    all variables preceded by a dot belong to the
    class definition body, all other variables
    are strangers.

- Class configuration for data types and classes

    A class is first of all, a data structure and computed
    representation of data and situational concepts to illustrate
    a real world situation. Looking at an object's structure,
    we can see that it has so many useful components we can explore, such as:

  - Parents:

    An object is certainly the result of it's preceding state. If it doesn't, it means it's a new concept at new sight. In onyx, we use the `#parents(...)` syntax to enumerate and collect it's preceding components. When used, this means that the child class heritates the methods, the attributes and all possible other characteristics of its parents.

  - Exterior Factors:

    This is certainly more complex than other concepts
    to get. These are the outside variables that helps us to
    create the instance of an object, taken in account for the
    object to exist. Those vars become certainly the property of
    the object so created. They belong to the object. This is
    certainly done with the help of the `#from(...)` keyword, that
    initializes those variables in the object's constructor to
    make them it's property.

  - Methods:

    The same dot convention distinguishes members from methods. An
    instance method is introduced by a dot, like an attribute, and is
    called on a receiver: `receiver.method(arguments)`. Inside its body,
    the receiver's members keep the same leading dot (`.attribute`,
    `.method()`), and `self` names the receiver only when it is passed as
    a value; the remaining parameters are those written in the signature.
    A class method is introduced by `cls.`; it is called on the class
    itself, `Class.method(arguments)`, and reaches class members through
    `cls.` the same way, typically as a construction helper.

  - Format:

    All the objects we use in a situation gives us information on
    how useful they can be in a situation. They possess attributes
    and that is why they appear in a given way. This been said, it
    suggests that these informations can be represented to us the
    way we want to see them. That's why we use `#format = "..."` to
    express our class's implementation's representation, with there
    is to be known by the language.

  - The IMPORTANT concept:

    In the `#from` operator, it is important to note that, if a `*` is
    preceding a variable, this means, that the concerned variables
    are required to create a class. If no default value is attributed
    to the other vars, the program will automatically raise an error and not compile (or interpret).
