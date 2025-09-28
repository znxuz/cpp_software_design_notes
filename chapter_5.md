# guideline 19 - use strategy to isolate how things are done

## problem case

Inheritance to further derive the concrete shape classes based on different
`draw()` implementation: `OpenGLCircle`, `MetalSquare`

- not scalable: exponential increase of the number of derived classes for new
  functionalities to differentiate:
    - `OpenGLProtobufCircle`, `OpenGLBoostSerialCircle`, `OpenGLProtobufSquare`
      etc.

> Inheritance is rarely the answer

## strategy pattern

- dependency inject a strategy for `draw()` to use and call

```cpp
class DrawStrategy {
public:
  virtual void draw(const Circle& c /*, ... */) const = 0;
  virtual void draw(const Square& c /*, ... */) const = 0;
};

class Shape {
public:
  virtual ~Shape() = default;
  virtual void draw(/* ... */) const = 0;
  // ...
};

class Circle : public Shape {
public:
  explicit Circle(double radius, std::unique_ptr<DrawStrategy> drawer) /* ... */
  void draw(/* ... */) const override { drawer_->draw(*this, /* ... */); }
private:
  std::unique_ptr<DrawStrategy> drawer_;
  // ...
};
```

- tight coupling between `DrawStrategy` and all the concrete shape classes

## comparison to visitor

- visitor:
    - the general addition of operations as the variation point
    - easy to add new operations, not easy to add new types
- strategy:
    - a single function/functionality as the variation point
    - easy to add new types, not easy to add new operations

## templatized strategy

- template param instead of `std::unique_ptr`

```cpp
template<typename DrawCircleStrategy>
class Circle : public Shape {
public:
  explicit Circle(double radius, DrawCircleStrategy drawer) /* ... */
  void draw(/*some arguments*/) const override { drawer_(*this, /* ... */); }
  // ...
};
```

---

# guideline 20 - favor composition over inheritance

duh... (tight coupling, easy to misuse/abuse)

---

# guideline 21 - use command to isolate what things are done

```cpp
class CalculateCommand {
public:
  virtual ~CalculateCommand() = default;
  virtual int execute(int i) const = 0;
  virtual int undo(int i) const = 0;
};

class Add : public CalculateCommand {
public:
  explicit Add(int operand) /* ... */
  int execute(int i) const override { return i + operand_; }
  int undo(int i) const override { return i - operand_; }
  /* ... */
};

class Subtract : public CalculateCommand {
public:
  explicit Subtract(int operand) /* ... */
  int execute(int i) const override { return i - operand_; }
  int undo(int i) const override { return i + operand_; }
  /* ... */
};

class Calculator {
public:
  void compute(std::unique_ptr<CalculateCommand> cmd) {
    current_ = cmd->execute(current_);
    cmd_stack_.push(std::move(cmd));
  }
  void undo_last() {
    if (cmd_stack_.empty()) return;

    auto cmd = std::move(cmd_stack_.top());
    cmd_stack_.pop();
    current_ = cmd.undo(current_);
  }
  void clear() {
    current_ = 0;
    std::stack<std::unique_ptr<CalculateCommand>>{}.swap(cmd_stack_);
  }
  int result() const { return current_; }

private:
  std::stack<std::unique_ptr<CalculateCommand>> cmd_stack_;
  int current_{};
};
```

> from a structural POV, the strategy and command patterns are *identical*; the
> difference lies in the intent: the strategy specifies *how* things should be
> done, while the command specifies *what* should be done.

- `std::partition()`: the given predicate is a strategy
- `std::for_each()`: the given unary function is a command

---

# guideline 22 - prefer value semantics over reference semantics

- design pattern in GoF style: reference semantics via pointers/references
- modern c++ philosophy: value semantics

> prefer implementing design patterns via value semantics over reference
> semantics

---

# guideline 23 - prefer value-based implementations over strategy and command

Instead of strategy base class:

> `using DrawStrategy = std::function<void(const Circle&, /* ... */)>;`

## shortcomings of the `std::function` solution

- can only replace single virtual function
