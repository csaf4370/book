Constants
=========

{{#include ../links.md}}

Constants can be defined using the `const` keyword and are immutable.

Constants follow the same naming rules as [variables].

```rust,no_run
const x = 42;

print(x * 2);       // prints 84

x = 123;            // <- syntax error: cannot assign to constant
```

```rust,no_run
const x;            // 'x' is a constant '()'

const x = 40 + 2;   // 'x' is a constant 42
```


Manually Add Constant into Custom Scope
--------------------------------------

It is possible to add a constant into a custom [`Scope`] so it'll be available to scripts
running with that [`Scope`].

When added to a custom [`Scope`], a constant can hold any value, not just a literal value.

It is very useful to have a constant value hold a [custom type], which essentially acts
as a [_singleton_](../patterns/singleton.md).

```rust,no_run
use rhai::{Engine, Scope};

#[derive(Debug, Clone)]
struct TestStruct(i64);                                     // custom type

let mut engine = Engine::new();

engine
    .register_type_with_name::<TestStruct>("TestStruct")    // register custom type
    .register_get("value", |obj: &mut TestStruct| obj.0),   // property getter
    .register_fn("update_value",
        |obj: &mut TestStruct, value: i64| obj.0 = value    // mutating method
    );

let mut scope = Scope::new();                               // create custom scope

scope.push_constant("MY_NUMBER", TestStruct(123_i64));      // add constant variable

// Beware: constant objects can still be modified via a method call!
engine.consume_with_scope(&mut scope,
r"
    MY_NUMBER.update_value(42);
    print(MY_NUMBER.value);                                 // prints 42
")?;
```


Caveat &ndash; Constants Can be Modified via Rust
------------------------------------------------

A custom type stored as a constant cannot be modified via script, but _can_ be modified via
a registered Rust function that takes a first `&mut` parameter &ndash; because there is no way for
Rhai to know whether the Rust function modifies its argument!

By default, functions with a first `&mut` parameter registered via the `Engine::register_XXX` API
or directly into [modules] always allow constants to be passed to them. This is because using `&mut`
can avoid unnecessary cloning of a [custom type] value, even though it is actually not modified.

See the relevant section on `&mut` parameter usage in [_Methods_](method.md) for more details.

```rust,no_run
const x = 42;       // a constant

x.increment();      // call 'increment' defined in Rust with '&mut' first parameter
                    // must use 'Dynamic::is_read_only' to check if parameter is constant

x == 43;            // value of 'x' is changed!

fn double() {
    this *= 2;      // function doubles 'this'
}

let y = 1;          // 'y' is not constant and mutable

y.double();         // double it...

y == 2;             // value of 'y' is changed as expected

x.double();         // <- error: cannot modify constant 'this'

x == 43;            // value of 'x' is unchanged by script
```

### Pure plugin functions

When functions are registered as part of a [plugin], `&mut` parameters are protected from being
passed constant values.

By default, [plugin functions] that take a first `&mut` parameter disallow passing constants as that
parameter.

However, if a [plugin function] is marked with the `#[export_fn(pure)]` or `#[rhai_fn(pure)]` attribute,
it is considered _pure_ (i.e. assumed to not modify its arguments) and so constants are allowed.

### Implications on script optimization

Rhai _assumes_ that constants are never changed, even via Rust functions.

This is important to keep in mind because the script [optimizer][script optimization]
by default does _constant propagation_ as a operation.

If a constant is eventually modified by a Rust function, the optimizer will not see
the updated value and will propagate the original initialization value instead.

`Dynamic::is_read_only` can be used to detect whether a [`Dynamic`] value is constant or not
within a Rust function.