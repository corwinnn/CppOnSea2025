# Monadic Operations

🎥 [Talk link](https://www.youtube.com/watch?v=fyjJPwkVOuw&list=PL5XXu3X6L7jsyWJKlIdo9ZNujqATJEMWL&index=19)

---

## Motivation

Imperative code with many failure checks often looks like this:

```cpp
bool getIntCellValueNegative(Db db, Key key, Location location, bool& result)
{
    Element element;
    if (!getElement(db, key, element)) { return false; }

    CTable table;
    if (!getTable(element, table)) { return false; }

    int value;
    if (!getInt(table, value)) { return false; }

    result = (value < 0);
    return true;
}
```

This works but is **verbose and deeply nested**.
Starting with C++23, we can express this more clearly using **`std::expected`**.

---

## Using `std::expected`

```cpp
std::expected<bool, MyError> getIntCellValueNegative(Db db, Key key, Location location)
{
    return getElement(db, key)
        .and_then(getTable)
        .and_then([location](const Table& table) { return getCell(table, location); })
        .and_then(getInt)
        .transform(isNegative)
        .or_else(logFailure);
}
```

### Explanation

* **`and_then`**: applies the next operation only if the previous one succeeded.
* **`transform`**: applies a function to the successful value.
* **`or_else`**: handles or logs an error.
* The chain expresses *the happy path* linearly — no explicit `if` nesting.

---

## Compositional thinking: chaining functions

Imagine you have:

```cpp
double foo(int x);
std::string bar(double x);

int x = 1;
std::string result = bar(foo(x));
```

To *vectorize* this easily using ranges:

```cpp
std::vector<std::string> piped(const std::vector<int>& xs) 
{
    // This is a "plan" — no actions have been performed yet (lazy evaluation)
    auto output = xs
        | std::views::transform(foo)
        | std::views::transform(bar);

    return std::ranges::to<std::vector<std::string>>(output);
}
```

### Key idea

Monadic operations generalize **function composition** —
the same way `and_then`, `transform`, and `or_else` compose actions that may fail,
`ranges::views::transform` composes actions that apply lazily to collections.

---

## Flattening nested loops with range pipelines

Traditional approach:

```cpp
void CClassicLoop::compileAll(const std::vector<CProject>& projects)
{
    for (const auto& project : projects) {
        auto files = getFilesInProject(project);

        for (const auto& file : files) {
            auto diagnostics = compile(file);

            for (const auto& diagnostic : diagnostics) {
                printDiagnostic(diagnostic);
            }
        }
    }
}
```

Using **range-based monadic composition**:

```cpp
void CRangeMonad::compileAll(const std::vector<CProject>& projects)
{
    namespace vw = std::views;

    auto diagnostics = projects
        | vw::transform(getFilesInProject) | vw::join
        | vw::transform(compile)           | vw::join;

    std::ranges::for_each(diagnostics, printDiagnostic);
}
```

### Notes

* `vw::join` flattens nested ranges (like `flat_map` in functional languages).
* The intent — *compile all files from all projects and print all diagnostics* — is now clear and declarative.

---

## Choosing the First Success

Another monadic pattern is trying multiple fallbacks:

```cpp
ELanguage getStartupLanguageOpt()
{
    return getLanguageFromCommandLineOpt()
        .or_else(getLanguageFromRegistryOpt)
        .or_else(getLanguageFromEnvironmentOpt)
        .value_or(ELanguage::English);
}
```

### Explanation

* Each `or_else` is called **only if the previous one failed**.
* `value_or` provides a default value if *all options fail*.
* This expresses “try these sources in order” clearly and safely.

---
