# guideline 15 - design for the addition of types, or operations

- dynamic polymorphism at run-time for differentiating object types:
	- procedual via a `enum` member:
		- easy to *extend types* by just adding new enumerators
		- but afterwards all associated operations need to be fixed/modified
		- the primitive C-way of programming
	- OOP via `virtual`:
		- easy to *extend common operations* by adding more virtual functions
		- but afterwards all inherited types need to be fixed/modified
		- adding more operations results in modifications of all classes on the
		  inheritance hierarchy (more virtual functions)

> dynamic polymorphism: a choice between easily extending the type or operation

| programming paradigm | strength                      | weakness                      |
|----------------------|-------------------------------|-------------------------------|
| procedual            | addition of operations        | addition of polymorphic types |
| OOP                  | addition of polymorphic types | addition of operations        |

---

# guideline 16 - use visitor to extend operations

> visitor lets you define a new operation without changing the classes of the
> elements on which it operates

## example

```cpp
class ShapeVisitor {
public:
  virtual ~ShapeVisitor() = default;
  virtual void visit(const Circle&, /*...*/) const = 0;
  virtual void visit(const Square&, /*...*/) const = 0;
  // more for each concrete shape...
};
```

Implement operations based off of `ShapeVisitor`:

```cpp
class Draw : public ShapeVisitor {
public:
  void visit(const Circle& c, /*...*/) const override { /*...*/ }
  void visit(const Square& s, /*...*/) const override { /*...*/ }
  // more fore each concrete shape
};

class Rotate : public ShapeVisitor {
public:
  void visit(const Circle& c, /*...*/) const override { /*...*/ }
  void visit(const Square& s, /*...*/) const override { /*...*/ }
  // more fore each concrete shape
};

// extend by adding new 'operation' classes
```

Apply the visitor pattern via `accept()` member function by passing itself to
the given visitor object:

```cpp
class Shape {
public:
  virtual ~Shape() = default;
  virtual void accept(const ShapeVisitor& v) = 0;
  // ...
};

class Circle : public Shape {
public:
  explicit Circle(double radius) : radius_{radius} {}
  void accept(const ShapeVisitor& v) override { v.visit(*this); }
  //...
};

class Square : public Shape {
public:
  explicit Square(double side) : side_{side} {}
  void accept(const ShapeVisitor& v) override { v.visit(*this); }
  // ...
};
```

Use the visitor pattern:

```cpp
for (const auto& shape : shapes) {
  shape->accept(Draw{});
}
```

## shortcomings

- inflexible:
	- e.g. a translation operation is equivalent for all shapes but the
	`visit()` function must still be implemented for each and every concrete
	shape
	- return type is fixed by the visitor base class: could be mitigated by
	  storing the result in the visitor to be accessed later
- easy to add new operations but hard to add new types:
	- modify the visitor base class
	- update all the visitor derived operation classes
- intrusive to implement/apply
- should mark the derived shape classes `final` in case extending from them but
  forgetting to implementing the `accept()` results in erroneous function being
  called
- *double dispatch*: two `virtual` functions called for just one operation
- rather complex to maintain

> also because of its cyclic dependency hierarchy, the visitor pattern requires
> a **closed set** of types in exchange for an **open set** of operations

---

# guideline 17 - consider `std::variant` for implementing visitor

```cpp
struct Print {
  void operator()(int value) const { std::println("int"); }
  void operator()(double value) const { std::println("double"); }
  void operator()(const std::string& value) const { std::println("string"); }
};

int main() {
  std::variant<int, double, std::string> v{};

  v = 42;
  v = 3.14;
  v = 2.71f;
  v = "hello";
  v = 43;
  // const int i = std::get<int>(v);
  // const int* p_i = std::get_if<int>(&v);

  std::visit(Print{}, v);
}
```

- `std::visit`: pass a custom **visitor** to perform any operation on the value
  of a *closed set* of types
- loose coupling: no `Visitor` base class, derived classes -> non intrusive

## refactored example

```cpp
class Circle {
public:
  explicit Circle(double radius) : radius_{radius} {}
  // ...
};

class Square {
public:
  explicit Square(double side) : side_{side} {}
  // ...
};

using Shape = std::variant<Circle, Square>;
using Shapes = std::vector<Shape>;
```

> just as POD classes

```cpp
struct Draw {
  void operator()(const Circle& c) const { /* ... */ }
  void operator()(const Square& c) const { /* ... */ }
};

void draw_all_shapes(const Shapes& shapes) {
  for (const auto& shape : shapes)
    std::visit(Draw{}, shape);
}

using RoundShapes = std::variant<Circle, Ellipse>;
using AngularShapes = std::variant<Square, Rectangle>;
```

- significantly simpler from the architectural POV

## benchmark

- 25k op on 10k randomly created shapes

| visitor impl.            | gcc 11.1 | clang 11.1 |
|--------------------------|----------|------------|
| classic visitor          | 1.61     | 1.80       |
| OOP                      | 1.52     | 1.14       |
| enum                     | 1.21     | 1.12       |
| std::variant             | 1.19     | 1.23       |
| std::variant w. get_if<> | 1.02     | 0.69       |

## shortcomings

- still for *open set* of operations on a *closed set* of types
- `std::variant` is the size of the biggest type: potential waste buffer space
- better performance for worse encapsulation

---

# guideline 18 - beware the performance of acyclic visitor pattern

> worst performance for supporting an open set of operations with an open set of
> types

skipped
