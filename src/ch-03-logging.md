# Logging

Teloxide uses the [`log`] facade crate.
It may be a good idea to add a logging implementation, so you can see logs from teloxide or use logging yourself.

[`log`]: https://lib.rs/crates/log

There are a lot of logging implementations (see [`log` documentation]) to choose from, but as a start you can try [`pretty_env_logger`].
For this add a dependency to `Cargo.toml`:

[`log` documentation]: https://docs.rs/log/latest/log/#in-executables
[`pretty_env_logger`]: https://lib.rs/crates/pretty_env_logger

```toml
pretty_env_logger = "0.4"
```

And call the init function somewhere near the beginning of the `main` function:

```rust,no_run
# use teloxide::prelude::*;
# #[tokio::main]
# async fn main() -> Result<(), Box<dyn std::error::Error>> {
pretty_env_logger::init();
#     let bot = Bot::new("TOKEN").auto_send();
# 
# teloxide::repl(bot, |message: Message, bot: AutoSend<Bot>| async move {
#     if let Some(text) = message.text() {
#         bot.send_message(message.chat.id, text).await?;
#     }
# }).await;
#  
#     Ok(())
# }
```

And that's all! You should be able to see logs:

![Output of `RUST_LOG=info cargo run`](./img/ch-03-pic-01.png)
