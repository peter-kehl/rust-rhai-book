`print` and `debug`
===================

{{#include ../links.md}}

The `print` and `debug` functions default to printing to `stdout`, with `debug` using standard debug formatting.

```js
print("hello");         // prints "hello" to stdout

print(1 + 2 + 3);       // prints "6" to stdout

let x = 42;

print(`hello${x}`);     // prints "hello42" to stdout

debug("world!");        // prints "world!" to stdout using debug formatting
```


Override `print` and `debug` with Callback Functions
----------------------------------------------------

When embedding Rhai into an application, it is usually necessary to trap `print` and `debug` output
(for logging into a tracking log, for example) with the `Engine::on_print` and `Engine::on_debug` methods.

```rust
// Any function or closure that takes an '&str' argument can be used to override 'print'.
engine.on_print(|x| println!("hello: {x}"));

// Any function or closure that takes a '&str', an 'Option<&str>' and a 'Position' argument
// can be used to override 'debug'.
engine.on_debug(|x, src, pos| {
    let src = src.unwrap_or("unknown");
    println!("DEBUG of {src} at {pos:?}: {s}")
});

// Example: quick-'n-dirty logging
let logbook = Arc::new(RwLock::new(Vec::<String>::new()));

// Redirect print/debug output to 'log'
let log = logbook.clone();
engine.on_print(move |s| {
    let entry = format!("entry: {}", s);
    log.write().unwrap().push(entry);
});

let log = logbook.clone();
engine.on_debug(move |s, src, pos| {
    let src = src.unwrap_or("unknown");
    let entry = format!("DEBUG of {src} at {pos:?}: {s}");
    log.write().unwrap().push(entry);
});

// Evaluate script
engine.run(script)?;

// 'logbook' captures all the 'print' and 'debug' output
for entry in logbook.read().unwrap().iter() {
    println!("{entry}");
}
```


`on_debug` Callback Signature
-----------------------------

The function signature passed to `Engine::on_debug` takes the following form.

> ```rust
> Fn(text: &str, source: Option<&str>, pos: Position)
> ```

where:

| Parameter |      Type      | Description                                                     |
| --------- | :------------: | --------------------------------------------------------------- |
| `text`    |     `&str`     | text to display                                                 |
| `source`  | `Option<&str>` | source of the current evaluation, if any                        |
| `pos`     |   `Position`   | position (line number and character offset) of the `debug` call |

The _source_ of a script evaluation is any text string provided to an [`AST`] via `AST::set_source`.

```admonish tip.small

If a [module] is loaded via an [`import`] statement, then the _source_ of functions defined within
the module will be the module's _path_.
```
