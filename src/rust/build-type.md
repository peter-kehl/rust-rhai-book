Register a Custom Type via the Type Builder
===========================================

{{#include ../links.md}}

Sometimes it is convenient to package a [custom type]'s API (i.e. [methods],
[properties][getters/setters], [indexers] and [type iterators]) together such that they can be more
easily managed.

This can be achieved by manually implementing the `CustomType` trait, which contains only a single method:

> ```rust
> fn build(builder: TypeBuilder<T>)
> ```

The `TypeBuilder` parameter provides a range of convenient methods to register [methods], property
[getters/setters], [indexers] and [type iterators] of a [custom type]:

| Method                 | Description                                                               |
| ---------------------- | ------------------------------------------------------------------------- |
| `with_name`            | set a friendly name                                                       |
| `on_print`             | register the [`to_string`] function that pretty-prints the [custom type]  |
| `on_debug`             | register the [`to_debug`] function that debug-prints the [custom type]    |
| `with_fn`              | register a [method] (or any function really)                              |
| `with_get`             | register a property [getter][getters/setters]                             |
| `with_set`             | register a property [getter][getters/setters]                             |
| `with_get_set`         | register property [getters/setters]                                       |
| `with_indexer_get`     | register an [indexer] get function                                        |
| `with_indexer_set`     | register an [indexer] set function                                        |
| `with_indexer_get_set` | register [indexer] get/set functions                                      |
| `is_iterable`          | automatically register a [type iterator] if the [custom type] is iterable |

```admonish tip.small "Tip: Use plugin module if starting from scratch"

The `CustomType` trait is typically used on external types that are already defined.

To define a [custom type] and implement its API from scratch, it is more convenient to use a [plugin module].
```


Example
-------

```rust
// Custom type
#[derive(Debug, Clone, Eq, PartialEq)]
struct Vec3 {
    x: i64,
    y: i64,
    z: i64,
}

// Custom type API
impl Vec3 {
    fn new(x: i64, y: i64, z: i64) -> Self {
        Self { x, y, z }
    }
    fn get_x(&mut self) -> i64 {
        self.x
    }
    fn set_x(&mut self, x: i64) {
        self.x = x
    }
    fn get_y(&mut self) -> i64 {
        self.y
    }
    fn set_y(&mut self, y: i64) {
        self.y = y
    }
    fn get_z(&mut self) -> i64 {
        self.z
    }
    fn set_z(&mut self, z: i64) {
        self.z = z
    }
}

// The custom type can even be iterated!
impl IntoIterator for Vec3 {
    type Item = i64;
    type IntoIter = std::vec::IntoIter<Self::Item>;

    fn into_iter(self) -> Self::IntoIter {
        vec![self.x, self.y, self.z].into_iter()
    }
}

// Use 'CustomType' to register the entire API
impl CustomType for Vec3 {
    fn build(mut builder: TypeBuilder<Self>) {
        builder
            .with_name("Vec3")
            .with_fn("vec3", Self::new)
            .is_iterable()
            .with_get_set("x", Self::get_x, Self::set_x)
            .with_get_set("y", Self::get_y, Self::set_y)
            .with_get_set("z", Self::get_z, Self::set_z)
            // Indexer get/set functions that do not panic on invalid indices
            .with_indexer_get_set(
                |vec: &mut Self, idx: i64) -> Result<i64, Box<EvalAltResult>> {
                    match idx {
                        0 => Ok(vec.x),
                        1 => Ok(vec.y),
                        2 => Ok(vec.z),
                        _ => Err(EvalAltResult::ErrorIndexNotFound(idx.Into(), Position::NONE).into()),
                    }
                },
                |vec: &mut Self, idx: i64, value: i64) -> Result<(), Box<EvalAltResult>> {
                    match idx {
                        0 => vec.x = value,
                        1 => vec.y = value,
                        2 => vec.z = value,
                        _ => Err(EvalAltResult::ErrorIndexNotFound(idx.Into(), Position::NONE).into()),
                    }
                    Ok(())
                }
            );
    }
}

let mut engine = Engine::new();

// Register the custom type in one go!
engine.build_type::<Vec3>();
```

~~~admonish question "TL;DR: Why isn't there `is_indexable`?"

Technically speaking, `TypeBuilder` can automatically register an [indexer] get function if the [custom type] implements `Index`.
Similarly, it can automatically register an [indexer] set function for `IndexMut`.

In practice, however, this is usually not desirable because most `Index`/`IndexMut` implementations panic on invalid indices.

For Rhai, it is necessary to handle invalid indices properly by returning an error.

Therefore, in the example above, the `with_indexer_get_set` method properly handles invalid indices by returning errors.
~~~
