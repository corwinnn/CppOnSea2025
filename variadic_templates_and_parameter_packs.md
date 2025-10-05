# Variadic Templates and Parameter Packs

🎥 [Talk link](https://youtu.be/zx4f7OT7Uec?si=AcCMrSP_vxtQ2-SU)

After C++11, we gained **variadic templates** and **parameter packs**.

```cpp
template <typename... Ts>
class Foo {};

template <typename... Ts>
auto foo(Ts... args) -> void {}

// Ts is a template/function parameter pack 
```

---

## Parameter Packs

* A pack can have **0 or more elements**.
* Must be **explicitly expanded** inside the body of a class or function.

### Example

```cpp
template <typename... Ts>
struct tuple_wrapper {
public:
    // Expanded into template argument list
    using storage_t = std::tuple<Ts...>;

    // Function parameter list
    tuple_wrapper(Ts... args) : storage(args...) {}

private:
    storage_t storage;
};

template <typename... Ts>
constexpr auto make_tuple_wrapper(Ts&&... args) -> tuple_wrapper<Ts...> {
    // std::forward is applied to each element in the pack
    return tuple_wrapper<Ts...>{ std::forward<Ts>(args)... };
}

// Size of the pack
// sizeof...(Ts)
```

---

### Conversion to a common type using an initializer list

*(Function can be constexpr since C++14)*

```cpp
template <typename T, typename... Ts>
constexpr auto sum(T init_val, Ts... args) {
    for (const auto i : {T{args}...}) {
        init_val += i;
    }
    return init_val;
}

static_assert(sum(1, 2, 3) == 6);
```

---

### Using the comma operator

Trick to perform an operation on each element in the pack.
No common type is required — operations are evaluated and discarded.

```cpp
template <typename... Ts>
auto print_args(Ts... args) {
    (void)std::initializer_list<int>{ (std::cout << args << "\n", 0)... };
}
```

---

### Lambda capture (C++20)

```cpp
[&args...] { do_something(args...); }
[...args = std::move(args)] { do_something(std::move(args)...); }
```

---

### `alignas` with parameter packs

Expands all `alignas` specifiers for the given types.

```cpp
alignas(Ts...) std::byte buffer[std::max({sizeof(Ts)...})];
```

---

## Fold Expressions (C++17)

Before fold expressions, summing values required recursion:

```cpp
template <typename H>
constexpr auto sum(H head) -> H {
    return head;
}

template <typename H, typename... T>
constexpr auto sum(H head, T... tail) -> H {
    return head + sum(tail...);
}
```

Now simplified with a **fold expression**:

```cpp
template <typename... Ts>
constexpr auto sum(Ts... args) {
    return (args + ...);
}
```

---

### Identity values for empty packs

| Operator | Default Value |   
| -------- | ------------- |
| `&&`     | `true`        |
| `||`     | `false`       |
| `,`      | `void()`      |

---

### Fold expressions with comma operator

```cpp
(std::filesystem::remove(args), ...);
```

---

### Complex logic in fold expressions

```cpp
template <typename F, typename... Ts>
constexpr auto find_first_if(F f, Ts... args) {
    std::optional<std::common_type_t<Ts...>> result;
    (void)((f(args) ? (result = args, true) : false) || ...);
    return result;
}
```

---

### Storing types

```cpp
using types = type_list<Ts...>;
```

---

### Abbreviated form (C++20)

```cpp
auto print(auto... args) -> void {
    // ...
}
```

---

## C++26 and Variadic Templates (Proposed)

### Indexing

```cpp
using first_type = Ts[0];
using last_type  = Ts[sizeof...(Ts) - 1];

const auto first_value = args...[0];
const auto last_value  = args[sizeof...(args) - 1];
```

---

### Structured Bindings for Variadic Expansion

```cpp
auto print_fields(const auto& s) {
    const auto& [...items] = s;
    (std::print("{} ", items), ...);
}

struct some_struct {
    int a;
    float b;
    std::string c;
};

some_struct s {1, 2.0f, "hello"};
print_fields(s);
```
