# pyo3_example
A small tutorial on how to make Rust -> Python bindings with Pyo3 and maturin.

This requires nightly rust, to set it up run:
```bash
rustup install nightly
```

In this example we have defined a simple function that sums two numbers in rust.
```rust
use pyo3::prelude::*;

#[pyfunction()]
#[text_signature = "(a, b)"]
/// Return the sum of the two input integers
///
/// Arguments
/// ---------
/// a: int,
///     The first integer addend
/// b: int
///     The second integer addend
pub fn add_two_numbers(a: u64, b: u64) -> u64 {
    a + b
}
```

In order to be able to call the function we must export it inside a module. The root module **has to** have the same name of the crate.
```rust
use pyo3::wrap_pyfunction;

#[pymodule]
fn simple_pkg(_py: Python, m: &PyModule) -> PyResult<()> {
    m.add_wrapped(wrap_pyfunction!(add_two_numbers))?;
    Ok(())
}
```

To make it compile we should modify the `Cargo.toml`, adding the following lines to use `Pyo3` (our bindings library):
```toml
[dependencies.pyo3]
version = "0.13.2"
# This settings allows the compiled wheel to work for any
# python version >= 3.6.
# While this greatly improves compatibility, it will
# generate slightly slower packages because we can only
# use a limited set of APIs
features = ["extension-module", "abi3-py36"]
```
Moreover, we need to specify the name of our python library and
that we want a `cdynlib` (a C compatible shared library) so that it is importable from python:
```toml
[lib]
name = "simple_pkg"
crate-type = ["cdylib"]
```
To build the library we can run
```bash
cargo run --release
```
and the resulting shared library will be in `./target/release/libsimple_pkg.so` (on Linux and Mac).

to use the library you have to rename it removing the `lib` prefix:
```bash
cp ./target/release/libsimple_pkg.so ./simple_pkg.so
```
Then if you are in the same folder you can import it in python:
```python
import simple_pkg

print(simple_pkg.add_two_numbers(1, 2))
# 3
```

To make and installable wheel install `maturin`:
```bash
cargo install maturin
```
and then for local development you can build a debug version and install it with:
```bash
maturin develop
```
otherwise to make a publishable version run:
```bash
maturin build
```
which will create the wheel in `./target/wheels/simple_pkg-0.1.0-cp36-abi3-<YOUR_PLATFORM_TAG>.whl`.

If you want to be able to create portable linux wheels you have to use a `ManyLinux2010` docker image.