% cbindgen

Alexis Beingessner

[<img src="icon.png" width="250" style="display:inline; box-shadow:none;"></img>](http://cglab.ca/~abeinges)
[<img src="rust.png" width="250" style="display:inline; box-shadow:none;"></img>](https://rust-lang.org)
<img src="firefox-nightly.png" width="250" style="display:inline; box-shadow:none;"></img>

slides: https://github.com/Gankro/tutorial-cbindgen/



# What is cbindgen?

Rust Library => C(++) Header

github.com/eqrion/cbindgen

```
cargo install cbindgen
```




# **Why** is cbindgen?

Webrender!



# Functions

```rust
#[no_mangle]
pub extern fn double(input: Option<&mut u32>) -> u32 { ... }
```

---

```cpp
extern "C" {
  uint32_t double(uint32_t* input);
}
```


# Structs/Unions

```rust
#[repr(C)]
pub struct MyStruct { a: u32, b: bool }
```

-------

```cpp
struct MyStruct { uint32_t a; bool b; };
```



# Tuple Structs

```rust
pub struct MyTuple(u8, MyStruct);
```

-------

```cpp
struct MyTuple { uint8_t _0; MyStruct _1; };
```


# Typedefs

```rust
pub type Weight = f32;
```

---------

```cpp
using Weight = float;
```


# Fieldless Enums


```rust
#[repr(u32)]
pub enum MyEnum { A, B, C }
```

-------

```cpp
enum class MyEnum: uint32_t { A, B, C }
```

# Fieldfull Enums

```rust
#[repr(u32)]
pub enum COptionU32 { Some(u32), None, }
```

-------

```cpp
union COptionU32 {
  enum class Tag : uint32_t { Some, None, };

  struct Some_Body { Tag tag; uint32_t _0; };

  struct { Tag tag; };
  Some_Body some;
};
```

# Fieldfull Enum Conveniences

```cpp
auto val = COptionU32::Some(12);
if (val.IsSome()) {
  printf("%d\n", val.some._0);
}
```


# Generic Structs (templated)

(not functions or enums)

```rust
#[repr(C)]
pub struct MyGenericStruct<T> {
    vals: [T; 16],
}
```

-------

```cpp
template<typename T>
struct MyGenericStruct {
  T vals[16];
};
```





# Generic Structs (instantiated)

```rust
#[no_mangle]
pub extern fn process(input: MyGenericStruct<u8>)  { ... }
```

----

```c
typedef struct {
  uint8_t vals[16];
} MyGenericStruct_u8;
```





# How Does it Work?

* grab all `#[no_mangle] pub extern fns`
* grab all types those reference
  * and types those reference
    * and ...


(note: hacky parser)



# How Does it Work?

```rust
#[no_mangle]
pub extern fn process(val: Input) -> bool { ... }

#[repr(C)]
pub struct Input {
  data: Data,
}

#[repr(u32)]
pub enum Data { A, B, C }
```



# How To Use It

* as a lib in build.rs
* as a cli tool




# Let's Look at a Simple Codebase

https://github.com/Gankro/tutorial-cbindgen

* `rust-bindings/`
  * `cbindgen.toml`
  * `Cargo.toml`
  * `src/lib.rs`
* `build.sh`
* `main.cpp`




# Pitfalls

* Doesn't respect namespacing
* Doesn't support non-POD types (destructors)
* Doesn't work if there's any `repr(rust)` types