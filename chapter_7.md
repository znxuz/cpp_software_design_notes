# guideline 28 - build bridges to remove physical dependencies

> aims to solve the transitive, invisible dependency $A -> B -> C ==> A -> C$

## approach 1

> using an base class abstraction

### `<engine.hpp>`

```cpp
class Engine {
public:
  virtual ~Engine() = default;
  virtual void start() = 0;
  virtual void stop() = 0;
  // ...

private:
  // ...
};
```

### `<electric_engine.hpp>`

```cpp
#include <engine.hpp>

class ElectricEngine : public Engine {
public:
  void start() override;
  void stop() override;
  // ...

private:
  // ...
};
```

### `<electric_car.hpp>`

```cpp
#include <engine.hpp> // just include the engine base class

class ElectricCar {
public:
  ElectricCar(/* ... */);
  void drive();
  // ...

  private:
  std::unique_ptr<Engine> engine_;
  // ...
};
```

### `<electric_car.cpp>`

```cpp
#include <electric_car.hpp>
#include <engine.hpp> // include the derived/concrete class header here

ElectricCar::ElectricCar(/* ... */)
    : engine_{ std::make_unique<ElectricEngine>(/* ... */) } /* ... */
{}
// ...
```

- the `Engine` base class abstraction serves as a *bridge* to decouple the car
  implementation from the actual engine implementation
- the `ElectricCar` header remains unchanged in case of modifications to either
  the engine or itself

> bridge isn't about *bringing the classes closer together* - on the contrary,
> it's about **separating concerns** and **loose coupling**, so that two
> entities can vary independently

## the pimpl idiom

- a simple bridge design pattern implementation

### `<person.hpp>`

```cpp
class Person {
public:
  // ...

private:
  struct Impl;
  const std::unique_ptr<Impl> pimpl_;
};
```

### `<person.cpp>`

```cpp
#include <person.hpp>

struct Person::Impl {
  std::string forename;
  std::string surname;
  std::string address;
  std::string city;
  std::string country;
  std::string zip;
  // ...
};
```

> encapsulate/decouple the actual implementation from the interface

- modifications doesn't affect the exposed interface

---

# guideline 29 - be aware of bridge performance gains and losses

> pointer indirection isn't free -> ca. 11%/13% gcc/clang performance penalty
> based on benchmark

## partial pimpl

- some members stay as direct members, others get moved into the pimpl
- performance increase by about 14%/6.5% gcc/clang, because of the partial size
  reduction via pimple

> but always profile to be sure

---

# guideline 30 - apply prototype for abstract copy operations
