# трюки в Rust

- [pipe](#pipe)
- [вместо замыканий](#вместо-замыканий)

## pipe

```rust
pub(crate) trait Also: Sized {
    fn also(mut self, f: impl FnOnce(&mut Self)) -> Self {
        f(&mut self);
        self
    }
}

impl<T> Also for T {}

// let x = 1.also(|x| x+1);
```


### turbo-fish

```rust
.closure_consumer(<_>::some_method)

//например
.map(<_>::to_string)
```

[наверх](#команды)
