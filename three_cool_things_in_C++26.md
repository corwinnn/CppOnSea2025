# Three Cool Things in Modern C++

## 1. C++26 — **Erroneous Behaviour (EB)**

| Concept                                         | Meaning                                                                                                            |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **UB — Undefined Behaviour**                    | *Anything can happen*, no guarantees. Compiler is free to optimize under the assumption "this never happens."      |
| **EB — Erroneous Behaviour** *(C++26)* | Well-defined as **wrong**, but **predictable and detectable**. Implementation inserts known “error poison values.” |

### Example: Reading Uninitialized Variables Becomes EB

In **C++26**, reading an uninitialized local variable **will no longer be UB**.
Instead, the compiler inserts a **poison error pattern**. This is **detectable**, has **minimal overhead**, and can be disabled.

```cpp
// Current behaviour (C++23 and earlier)
auto f1() {
    // Fill stack
    char a[] = { 's', 'e', 'c', 'r', 'e', 't' };
} // Stack pointer moves but memory is untouched

auto f2() {
    char a[6];   // Stack pointer reclaims same memory
    print(a);    // Might print "secret" by accident — UB, optimizer free to ignore it
}

int main() {
    f1();
    f2(); // In practice prints "secret", but compiler is allowed to do anything
}
```

**In C++26 with EB mode**: `a` will be filled with known **"error pattern" bytes**, preventing leakage of stale memory content.

### Other EB-Related Improvements

* **Standard library bounds checking** for all sequence containers (`std::vector`, `std::array`, etc.)
  → When out-of-bounds access is detected, implementation can inject error-semantics instead of UB.
* **Trivial Relocation** — faster than `std::move`, relocates raw bits **with awareness of lifetimes**.

---

## 2. **Compile-Time Reflection**

> **The program can "see" its own structure and generate code — at compile-time.**

* No runtime cost.
* Enables **code generation** inside the compiler pipeline.
* Replaces external tools like **Qt moc**, **C++/CLI**, **C++/CX**, IDL compilers, etc.

### Goal (C++29 Evolving Proposal)

Generate this automatically:

```cpp
class IFoo {
public:
    virtual int f() = 0;
    virtual void g(std::string) = 0;

    virtual ~IFoo() = default;
    IFoo() = default;
    IFoo(const IFoo&) = delete;
    void operator=(const IFoo&) = delete;
};
```

From this declaration:

```cpp
class(interface) IFoo {
    int f();
    void g(std::string);
};
```

> Today we **cannot** emit code in the same translation unit automatically
> In **C++26**, however, we **can reflect and output** metadata to another file

Example (valid in proposed reflection TS syntax):

```cpp
namespace __proto {
    class Widget {
        int f();
        void g(std::string);
    };
}

int main() {
    std::cout << interface(^^__proto::Widget); // Emits interface boilerplate
}
```

---

### **Type Category Concepts (Reflection Aware)**

| Category           | Meaning                                                                            |
| ------------------ | ---------------------------------------------------------------------------------- |
| `interface`        | Abstract class with only pure virtual functions                                    |
| `polymorphic_base` | Non-copyable/movable polymorphic base, with protected or public virtual destructor |
| `ordered`          | Type with `<=>` implementing `std::strong_ordering`                                |
| `copyable`         | Standard move/copy constructible and assignable                                    |
| `basic_value`      | `copyable` with default constructor and destructor, no protected/virtual members   |
| `value`            | `basic_value` + ordered semantics                                                  |
| `struct`           | `basic_value`, all members public, no virtuals or custom assignment                |
| `enum`             | `basic_value` with discrete named values                                           |
| `flag_enum`        | `enum` + supports bitwise operations                                               |
| `union`            | Tagged union style (safer alternative to raw C union)                              |

---

### Real Reflection Use Cases

*  JSON serializer/deserializer auto-generation
*  Automatic language bindings (Python, JS, C#...)
*  Binary metadata generation
*  Replace Qt moc / COM / CX / IDL code generators
*  Generate editor bindings, inspector panels, property sheets

---

## 3. New Async Pipeline Model (C++26 Executors)

### Current C++ async style

```cpp
future<int> res = pool.run([=]{ return f(x); });
// code here runs potentially concurrently with f
use_result(res.get());
```

### **C++26 Execution Pipeline Model**

```cpp
auto work = schedule(pool)
          | then([=]{ return f(x); });
// code here runs potentially concurrently with f
auto res = sync_wait(work).value();
```

> Think of it as **ranges but for execution** — a declarative pipeline of asynchronous tasks.

---

## Summary — Three Emerging Powers in C++26+

| Feature                              | Purpose                                                                |
| ------------------------------------ | ---------------------------------------------------------------------- |
| **Erroneous Behaviour**              | Make failures *detectable and defined*, improve debug workflow         |
| **Reflection-Based Code Generation** | Remove external preprocessors and enable compile-time meta-programming |
| **Async Pipelines**                  | Treat execution like ranges — composable, lazy, structured             |

---
