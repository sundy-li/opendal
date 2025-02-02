# OpenDAL

Open **D**ata **A**ccess **L**ayer that connect the whole world together.

## Status

OpenDAL is in **alpha** stage and has been early adopted by [databend](https://github.com/datafuselabs/databend/). Welcome any feedback at [Discussions](https://github.com/datafuselabs/opendal/discussions)!

## Quickstart

```rust
use anyhow::Result;
use futures::AsyncReadExt;
use futures::StreamExt;
use opendal::services::fs;
use opendal::ObjectMode;
use opendal::Operator;

#[tokio::main]
async fn main() -> Result<()> {
    let op = Operator::new(fs::Backend::build().root("/tmp").finish().await?);

    let o = op.object("test_file");

    // Write data info file;
    let w = o.writer();
    let n = w
        .write_bytes("Hello, World!".to_string().into_bytes())
        .await?;
    assert_eq!(n, 13);

    // Read data from file;
    let mut r = o.reader();
    let mut buf = vec![];
    let n = r.read_to_end(&mut buf).await?;
    assert_eq!(n, 13);
    assert_eq!(String::from_utf8_lossy(&buf), "Hello, World!");

    // Get file's Metadata
    let meta = o.metadata().await?;
    assert_eq!(meta.content_length(), 13);

    // List current dir.
    let mut obs = op.objects("").map(|o| o.expect("list object"));
    while let Some(o) = obs.next().await {
        let meta = o.metadata().await?;
        if meta.path().contains("test_file") {
            let mode = meta.mode();
            assert!(mode.contains(ObjectMode::FILE));
        }
    }

    // Delete file.
    o.delete().await?;

    Ok(())
}
```

## Contributing

Check out the [CONTRIBUTING.md](./CONTRIBUTING.md) guide for more details on getting started with contributing to this project.

## License

OpenDAL is licensed under [Apache 2.0](./LICENSE).
