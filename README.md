# hotswap

[![Crates.io][crates_badge]][crate]
[![Linux Build Status][travis_badge]][Travis CI]
[![Windows Build Status][appveyor_badge]][Appveyor CI]

[crates_badge]: https://img.shields.io/crates/v/hotswap.svg
[travis_badge]: https://travis-ci.org/draivin/rust-hotswap.svg?branch=master
[appveyor_badge]: https://ci.appveyor.com/api/projects/status/github/draivin/rust-hotswap?svg=true&branch=master

[crate]: https://crates.io/crates/hotswap
[Travis CI]: https://travis-ci.org/draivin/rust-hotswap
[Appveyor CI]: https://ci.appveyor.com/project/draivin/rust-hotswap

A library for hotswapping running code with minimal effort, requires a nightly
rust build.

Beware that the library is a prototype for now, and it may crash frequently.

## Usage
- Add the `hotswap` and `hotswap-runtime` dependencies to your `Cargo.toml`.
- Add a `dylib` build with the same project name and path to your `Cargo.toml`.
- Add the `#![feature(plugin, const_fn, custom_inner_attributes)]` feature gates.
- Import the plugin `#![plugin(hotswap)]`.
- Annotate the functions you want to hotswap with the `#[hotswap]` modifier.
- Add `#![hotswap_header]` attribute to the top of your program.
- Add `unsafe { hotswap_start!() }` to the entry point of your program, before
  you call any hotswapped functions.

## Current Limitations
- Changing hotswapped function signatures **WILL** result in a segfault.
  - Maybe this can be fixed by storing the types as metadata.
- Requires user code to use some non-local feature gates.

## Example
```toml
# Cargo.toml

[package]
name = "hotswapdemo"
version = "0.1.0"

[lib]
# This must be the same as the package name (with hyphens replaced with
# underscores). Anything else will cause an error at runtime.
name = "hotswapdemo"
crate-type = ["dylib"]
path = "src/main.rs"

[dependencies]
hotswap = "*"
hotswap-runtime = "*"
```

```rust
// main.rs

#![feature(plugin, const_fn, custom_inner_attributes)]
#![plugin(hotswap)]
#![hotswap_header]

use std::thread::sleep;
use std::time::Duration;

#[hotswap]
fn test(test: i32) -> () {
    println!("Foo: {}", test);
}

fn main() {
    unsafe { hotswap_start!() }

    let mut i = 1;
    loop {
        test(i);
        i += 1;
        sleep(Duration::from_millis(2000));
    }
}

```

That's it!

From there you can run the binary

```
> cargo run
     Running `target/debug/hotswapdemo`
Foo: 1
Foo: 2
Foo: 3
```

Then, once it is running, you can edit the printing code, e.g.

```rust
    println!("Bar: {} :)", test);
```

Once you recompile the code on another terminal (or on the same one using
background), you'll see the changes!

```
> cargo build --lib
   Compiling hotswapdemo v0.1.0 [...]
> fg
Foo: 7
Foo: 8
Bar: 9 :)
Bar: 10 :)
```

The running code will update without restarting the binary or losing state!

# See Also

* https://github.com/emoon/dynamic_reload
