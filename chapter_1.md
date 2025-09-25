# guideline 1 - understanding the importance of software design

Fix A -> involves B -> breaks C and D

> dependency is the key problem in software development at all scales

> reduce artificial dependencies through necessary abstractions and compromises

---

# guideline 2 - design for change

- separation of concerns
- module cohesion: force-separating the module would cause tight(artificial)
  coupling

> everything should just do one thing (like Unix-philosophy, linux wins again)

## example from the book

- in the example: method for serialization doesn't belong in a `Document` class
  (e.g. let a dedicated serializer with abilities like handling endianness/file
  format etc. do the job -> necessary abstraction)
- don't introduce pure virtual functions w/ added dependencies which aren't
  strictly needed by the sub-class -> artificial dependency

> classes with (external) artificial dependencies / strong couplings are hard to
> change, because a change would cause a chain of necessary modifications
> down-streamed to all the sub-classes

> artificial dependencies can even be transitive and carry over to the
> higher-level modules

## DRY - don't repeat yourself

- don't assume a change will never happen: case in point don't hardcode the 19%
  tax rate for each item class

---

# guideline 3 - separate interfaces to avoid artificial coupling

A function should take a interface class as argument rather than a concrete
class so that it is only exposed to a specific functionality.

> separate functionalities into extendable interfaces to minimize
> (artificial/exposed) dependencies

---

# guideline 4 - design for testability

## testability of private functions

- offload the call to testable public functions: extra dependency on public
  functions
- `friend` class declaration for test components: simple & quick, but introduces
  another dependency for production code
- `protected` instead of `private`: a non-solution
- `#define private public`: a god-send solution

> the true solution: as free functions instead of member functions for
> testability

- per Scott Meyers: *prefer nonmember, non-`friend` functions to member
  functions*
- increased encapsulation: because every member function has full access of
  internal components
- separate concerns for testability

> this is not an attack against (private) member/virtual functions: functions
> which are private and need to be tested fall into this scope

---

# guideline 5 - design for extension

## open-closed principle

### example from the book

- `document` class has:
	- pure virtual member function `serialize()`
	- an `enum` for different types of serialization: `pdf`, `word`
	- `serialize()` function uses the enum to differentiate
- problem: extending the `enum` means that all functions which depend on the
  enum need to be changed

> solution: extract the functionality for serialization

- the `document` class then becomes *open for extension, closed for
  modifications*
- modifications in the `serialization` component and the added abstraction from
  it is ok
