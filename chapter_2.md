# guideline 6 - adhere to expected behavior of abstractions

## liskov substitution principle (LSP)

> preconditions from the base type must not be further strengthened/weakened by
  the subtype

Thus a `square` is not a `rectangle` from an inheritance standpoint, because the
square introduces the invariant(a property that must always remain true for a
class to be considered valid/consistent) that all sides must be equal.

---

# guideline 7 - the similarities between base class and concepts

- c++20 concepts as static polymorphism
- LSP also applies to concepts

---

# guideline 8 - the semantic of requirements of overload sets

- overload sets: just means overloaded free functions
- the power of free functions:
	- a compile-time abstraction mechanism
	- base for generic programming

---

# guideline 9 - pay attention to ownership of abstractions

## dependency inversion principle

> depend on abstractions, not concrete types

- an implementation of dependency inversion: dependency injection

---

# guideline 10 - consider creating an architectural document

- agile, cicd
