# Namespaces in Modern C++

🎥 [Talk link](https://www.youtube.com/watch?v=uh9RWX-SXvc&list=PL5XXu3X6L7jsyWJKlIdo9ZNujqATJEMWL&index=35)

Namespaces are essential for **modularization**, **symbol isolation**, and **large-scale code organization**.

### Why use namespaces?

* Organize code into **logical units** (like folders in a filesystem)
* Improve **readability and modularity**
* Prevent **name clashes and redefinitions**
* Essential for **large and external/multi-module codebases**

---

## Inline Namespaces (C++11)

Used primarily for **versioning of libraries**.

```cpp
namespace lib 
{
    namespace v1 { void foo() {} }
    inline namespace v2 { void foo() {} }
}

int main() {
    lib::foo(); // Calls v2::foo — inline namespace is preferred implicitly
}
```

---

## Nested Namespaces

### C++17 shorthand syntax

```cpp
namespace outer::inner {
    void foo() {}
}
```

### Nested inline namespaces (C++20)

```cpp
namespace outer::middle::inline inner {
    void foo() {}
}
```

---

## Name Lookup

### ✅ Qualified Name Lookup (QNL)

Explicit lookup using `::`

* Scope is limited to **exactly** the namespace to the left of `::`
* Does **not** search outer scopes automatically

```cpp
namespace c {
    int x = 3;
}

namespace a::b {
    using namespace c;
    int x = 2;
}

a::b::x; // -> 2 (c::x is not considered because a::b defines its own x)
```

```cpp
namespace a {
    int x = 1;
    namespace b {};
}

a::b::x; // ❌ Compilation error — QNL does not check outer `a::`
```

---

### ✅ Unqualified Name Lookup (UNL)

Triggered when no `::` is used → searches:

1. Local scope
2. Enclosing scopes outward
3. Associated namespace (for ADL — see below)

```cpp
int x = 2;
int q = 3;

void func() {
    int x = 3;
    int y = x;  // UNL → 3 (local)
    int z = ::x; // QNL → 2 (global)
    int w = q;  // UNL → found in global scope
}
```

```cpp
int x = 2;
namespace c { int x = 3; }

namespace a::b {
    using namespace c;
    std::cout << x; // ❌ Ambiguous: ::x vs c::x
}
```

---

### ✅ Argument-Dependent Lookup (ADL)

Allows functions to be found based on the namespace of their **argument types**.

```cpp
namespace math {
    struct Vec {};
    void normalize(Vec) {}
    bool operator==(Vec, Vec) {}
}

void foo() {
    math::Vec a, b;
    normalize(a); // OK via ADL — math::normalize is found
    if (a == b) {} // OK — operator== found via ADL
    
    math::Vec c(a); // ✅ Class constructors do NOT use ADL
}
```

---

## Using Declarations vs Using Directives

* `using namespace X;` — brings **everything** from `X` (discouraged)
* `using X::name;` — brings **just one symbol** (preferred)

```cpp
namespace a { int x = 1; }
namespace b { int x = 2; }

int main() {
    using namespace a; // entire namespace
    using b::x;        // single symbol
    return x;          // -> 2 (single using-declaration has priority)
}
```

---

## Namespace Aliases

```cpp
namespace a::b::c {}
using abc = a::b::c;
```

Gives shorter access to deep namespace hierarchies.

---

## Anonymous (Unnamed) Namespaces

```cpp
namespace {
    void helper() {} // internal linkage — only visible in this translation unit
}
```

✅ Purpose:

* Provides **internal linkage**
* Prevents name collisions across TUs
* **Preferred to `static` functions/variables**

🚫 **Do NOT put unnamed namespaces in headers** — each include creates a separate entity → ODR violations.

---

## Best Practices (from Core Guidelines)

✔ Place helper functions in the **same namespace** as the class they support
✔ Place **operator overloads** in the namespace of their operands (for ADL)
✔ Use namespaces to express **logical code structure**, not just to avoid name collisions
✔ Prefer **`using X::symbol;` over `using namespace X;`**
✔ Avoid unnamed namespaces in **headers**

---
