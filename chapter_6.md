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


