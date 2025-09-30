# guideline 24 - use adapters to standardize interfaces

> just a wrapper class exposing a compatible API for "forwarding" calls to a
> non-compatible/standardized interface

```cpp
class Pages : public Document { // compatible object adapter/wrapper
public:
  void export_to_json() const override { export_to_json_format(pages /*..*/); }
  void serialize() const override { pages.convert_to_bytes(/*..*/); }
  // ...

private:
  OpenPages pages; // non compatible API
  // ...
};
```

## `begin()` and `end()`: examples as a function adapter

Via ADL, custom implemented `begin()` and `end()` can be supplied

```cpp
/* range-based for-loop unrolled */
using std::begin;
using std::end;
auto first{begin(range)};
auto last{end(range)};
for(; first != last; ++first) { /* ... */ }
```

- non-intrusive way to add  free function to any type

---

# guideline 25 - apply observers as an abstract notification mechanism

```cpp
template<typename Subject, typename StateTag>
class Observer {
public:
  virtual ~Observer() = default;
  virtual void update(const Subject& subject, StateTag property) = 0;
};

class Person {
public:
  enum StateChange {
    forename,
    surname,
    address
  };

  using PersonObserver = Observer<Person, StateChange>;

  explicit Person(std::string forname, std::string surname);

  bool attach(PersonObserver* observer) {
    auto [pos, success] = observers_.insert(observer);
    return success;
  }

  bool detach(PersonObserver* observer) { return observers_.erase(observer); }

  void notify(StateChange property) {
    for (auto iter = begin(observers_); iter != end(observers_);) {
      auto const pos = iter++;
      (*pos)->update(*this, property);
    }
  }

  // setters calling notify() with the appropriate enum after change

priavte:
  std::string forname_, surname_, address_;
  std::set<PersonObserver*> observers_;
};

class NameObserver : public Observer<Person, Person::StateChange> {
public:
  void update(const Person& person, Person::StateChange property) override {
    // handle & respond to changes
  }
};
```

## now based on value semantics

```cpp
template<typename Subject, typename StateTag>
class Observer {
public:
  using OnUpdate = std::function<void(const Subject&, StateTag)>;

  explicit Observer(OnUpdate on_update) on_update_{on_update} { /* ... */ }

  void update(const Subject& subject, StateTag property) {
    on_update_(subject, property);
  }

private:
  OnUpdate on_update_;
};
```

- pass the update-handling code directly as as a `std::function` in the ctor

## shortcomings

- works for single `update()` observers
- order or registration matters

---

# guideline 26 - user CRTP to introduce static type categories

- compile-time inheritance by upgrading the base class to a class template

## example

```cpp
template <typename Derived>
class Shape {
protected:
  ~Shape() = default; // prevents deletion through base pointer:
                      //     Shape<Circle>* p = new Circle(); delete p;
                      // allows: Circle* p = new Circle(); delete p;

public:
  Derived& derived() { return static_cast<Derived&>(*this); }
  const Derived& derived() const { return static_cast<Derived&>(*this); }

  void draw() const { derived().drawImpl(); }

  /*
  void draw() const {
    // choose one
    static_cast<const Derived*>(this)->drawImpl();
    static_cast<const Derived&>(*this).drawImpl();
  }
  */
};

class Circle : public Shape<Circle> {
public:
  void drawImpl() const { std::cout << "Drawing Circle\n"; }
};

class Square : public Shape<Square> {
public:
  void drawImpl() const { std::cout << "Drawing Square\n"; }
};

using ShapeVariant = std::variant<Circle, Square>;

int main() {
  std::vector<ShapeVariant> shapes;
  shapes.emplace_back(Circle{});
  shapes.emplace_back(Square{});

  for (const auto& shape : shapes) {
    std::visit(
        [](const auto& s) { s.draw(); /* calls the correct implementation */ },
        shape);
  }
}
```

- `protected` dtor also disables move operations
- use `decltype(auto)` when return type needs to come from the derived class:
	- keep cv-qualifiers & references unlike just `auto`

## shortcomings

- no common base class
- intrusive: everything becomes a template

## C++20 alternative to CRTP - concepts

```cpp
template<typename T>
concept DenseVector = 
  requires(T t, size_t index) {
    t.size();
    t.[index];
    { t.begin() } -> std::same_as<typename T::iterator>;
    { t.end() } -> std::same_as<typename T::iterator>;
  } &&
  requires(const T t, size_t index) {
    t[index];
    { t.begin() } -> std::same_as<typename T::iterator>;
    { t.end() } -> std::same_as<typename T::iterator>;
  };

template<DenseVector VectorType>
std::ostream& operator<<(std::ostream& os, const VectorType& vector) { /*..*/ }
```

> rather inflexible because the class need to match exactly: consider use a
> empty struct as tag

```cpp
struct DenseVectorTag {}:

template<typename T>
concept DenseVector = 
  // ... all requirements as before
  && std::is_base_of_v<DenseVectorTag, T>;

template<typename T>
class DynamicVector : private DenseVectorTag { /* ... */ };
```

---

# guideline 27 - use CRTP for static mixin classes

```cpp
template<typename T, typename Tag>
struct StrongType {
public:
  using value_type = T;
  explicit StrongType(const T& value) : value_(value) {}
  T& get() { return value_; }
  const T& get() const { return value_; }

private:
  T value_;
};

template<typename T>
using Meter = StrongType<T, struct MeterTag>;

template<typename T>
using Kilometer = StrongType<T, struct KilometerTag>;

using Surname = StrongType<std::string, struct SurnameTag>;
```

## problem

With a `operator+()` free function:

```cpp
template<typename T, typename Tag>
StrongType<T, Tag> operator+(const StrongType<T, Tag>& a,
                             const StrongType<T, Tag>& b) {
  return StrongType<T, Tag>(a.get() + b.get());
}
```

- only need the ability for adding/subtracting for certain types but all types
  are now "overloaded" with `operator+()`

> CRTP as an implementation pattern

```cpp
template<typename Derived>
struct Addable {
  friend Derived& operator+(Derived& lhs, Derived& rhs) {
    return Derived{lhs.get() + rhs.get()};
  }
  friend Derived& operator+=(Derived& lhs, Derived& rhs) {
    lhs.get() += rhs.get();
    return lhs;
  }
};

template<typename Derived>
struct Printable {
  friend std::ostream& operator<<(std::ostream& os, const Derived& d) {
    return os << d.get();
  }
};

template<typename Derived>
struct Swappable {
  friend void swap(Derived& lhs, Derived& rhs) {
    using std::swap;
    swap(lhs.get(), rhs.get());
  }
};

template<typename T, typename Tag, template<typename> class... Skills>
struct StrongType : private Skills<StrongType<T, Tag, Skills...>>...

template<typename T>
using Meter = StrongType<T, struct MeterTag, Addable, Printable, Swappable>;

template<typename T>
using KiloMeter = StrongType<T, struct MeterTag, Addable, Printable, Swappable>;

using Surname = StrongType<string, struct SurnameTag, Printable, Swappable>;
```

crazy syntax...
