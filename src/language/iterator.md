Iterator for Custom Type
========================

{{#include ../links.md}}

If a [custom type] is iterable, the [`for`] loop can be used to iterate through
its items in sequence.

In order to use a [`for`] statement, a _type iterator_ must be registered for
the [custom type] in question.

`Engine::register_iterator<T>` allows registration of a _type iterator_ for any type
that implements `IntoIterator`.

With a type iterator registered, the [custom type] can be iterated through.

```rust no_run
┌──────┐
│ Rust │
└──────┘

// Custom type
#[derive(Debug, Clone)]
struct TestStruct { fields: Vec<i64> }

// Implement 'IntoIterator' trait
impl IntoIterator<Item = i64> for TestStruct {
    type Item = i64;
    type IntoIter = std::vec::IntoIter<Self::Item>;

    fn into_iter(self) -> Self::IntoIter {
        self.fields.into_iter()
    }
}

// Register API and type iterator for 'TestStruct'
engine.register_type_with_name::<TestStruct>("TestStruct")
      .register_fn("new_ts", || TestStruct { fields: vec![1, 2, 3, 42] })
      .register_iterator::<TestStruct>();


┌─────────────┐
│ Rhai script │
└─────────────┘

let ts = new_ts();

// Use 'for' statement to loop through items in 'ts'
for value in ts {
    ...
}
```
