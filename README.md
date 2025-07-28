# SimpleDBNN

**SimpleDBNN** is a Rust library that combines a lightweight text database with a vector index for performing similarity searches using embeddings. It is designed to be flexible and embeddable with any custom embedding engine that implements the `Embeddable` trait.

## Features

- Efficient persistence with [`heed`](https://docs.rs/heed/) and [`arroy`](https://docs.rs/arroy/)
- Insert and query by embedding vectors
- Extensible interface via the `Embeddable` trait
- Batch support for high-throughput use cases
- Built-in test suite

## Installation

Add this to your `Cargo.toml`:

```toml
[dependencies]
simple_db_nn = "0.1"
```

> Note: replace with the published version on crates.io when available.

## 🚀 Usage

### 1. Define your embedding engine

```rust
struct DummyEmbedding;

impl Embeddable for DummyEmbedding {
    fn to_embedding(&self, content: Vec<u8>) -> Vec<f32> {
        let content_str = String::from_utf8(content).unwrap();
        if content_str.starts_with("$") {
            vec![100.0; 384]
        } else {
            vec![0.0; 384]
        }
    }
}
```

### 2. Create a database

```rust
use arroy::distances::Euclidean;
use simple_db_nn::{SimpleDBNN, Embeddable};
use std::path::PathBuf;

let mut db = SimpleDBNN::new(
    PathBuf::from("./db"),
    PathBuf::from("./embedded_db"),
    PathBuf::from("./config.json"),
    DummyEmbedding,
    384,
    0,
    42,
).unwrap();
```

### 3. Insert content

```rust
db.put("Hello world").unwrap();
```

### 4. Search similar content

```rust
let results = db.get("Hello", 4).unwrap();
for (id, dist, text) in results {
    println!("ID: {id}, Distance: {dist}, Text: {text}");
}
```

## 🧩 Public API

| Function               | Description                             |
| ---------------------- | --------------------------------------- |
| `put(&str)`            | Insert and index content                |
| `get(&str, usize)`     | Search top-n similar entries            |
| `put_batch(Vec<&str>)` | Insert a batch of entries               |
| `clear()`              | Delete all persisted data               |
| `get_current_id()`     | Get the next internal ID to be assigned |

## 🧠 `Embeddable` Trait

Implement this trait to integrate your own embedding engine:

```rust
pub trait Embeddable {
    fn to_embedding(&self, content: Vec<u8>) -> Vec<f32>;
}
```

## 🛠️ Tests

Run tests with:

```bash
cargo test
```

Covers:

- Basic storage and retrieval
- Batch insertions
- Similarity search using both `DummyEmbedding` and `FastEmbedding`

## ⚖️ License

This project is dual-licensed under MIT or Apache-2.0 — choose whichever you prefer.

## 🙌 Credits

- Powered by [`arroy`](https://crates.io/crates/arroy) for approximate nearest neighbor indexing
- Uses [`heed`](https://crates.io/crates/heed) for efficient LMDB-based storage
- Compatible with [`fastembed`](https://crates.io/crates/fastembed) for real embedding backends

---

Have a custom embedding engine? Just implement `Embeddable` and you're ready.

Pull requests and suggestions are welcome ❤️
