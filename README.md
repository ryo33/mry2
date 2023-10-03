# mry2

A simple mocking library.

**Status: Totally not implemented yet.**

## Usage

### 1. Let `Cat` be a struct with a method `meow`:

```rust
struct Cat {
    name: String,
}
```

### 2. Put `#[mry2::trait(Cat)]` on the impl block of `Cat`:

```rust
#[mry2::trait(Cat)]
impl Cat {
    fn meow(&self) -> String {
        format!("{}: meow", self.name)
    }
}
```

### 3. The expanded code:

```rust
struct CatObj(CatObjInner);

impl CatObj {
    pub fn new(cat: impl CatInterface) -> Self {
        CatObj(CatObjInner::Dyn(Box::new(cat)))
    }
}

struct CatObjInner {
    Real(Cat),
    Dyn(Box<dyn Cat>),
}

trait CatInterface {
    fn meow(&self) -> String;
}

impl Cat {
    fn meow(&self) -> String {
        format!("{}: meow", self.name)
    }
}

impl CatInterface for CatObj {
    fn meow(&self) -> String {
        match self.0 {
            CatObjInner::Real(cat) => cat.meow(),
            CatObjInner::Dyn(cat) => cat.meow(),
        }
    }
}

impl IntoObject for Cat {
    type Object = CatObj;
    fn obj(self) -> Self::Object {
        CatObj(CatObjInner::Real(self))
    }
}

#[derive(Clone, Default)]
struct MockCat { /* hidden */ };

impl CatInterface for MockCat {
    fn meow(&self) -> String {
        /* hidden */
    }
}

impl IntoObject for MockCat {
    type Object = CatObj;
    fn obj(self) -> Self::Object {
        CatObj(CatObjInner::Dyn(Box::new(self)))
    }
}
```

### 4. Use `CatObj` instead of `Cat`:

```rust
fn a_function_to_test(cat: CatObj) {
    cat.meow();
}
```

### 5. Use `MockCat` to mock `Cat`:

```rust
let mut mock_cat = MockCat::default();
/* set expectations on mock_cat */
a_function_to_test(mock_cat.obj());
```
