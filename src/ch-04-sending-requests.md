# Sending requests

We've already sent simple requests in previous chapters, but we didn't explain how they work.
In this chapter we'll dive deep into how requests are implemented in teloxide.

When in previous chapters we've called `send_message`, the method came from a trait called [`Requester`].
[`Requester`] is a trait that is implemented by [`Bot`] (and bot [adaptors], more on that in the next chapter) and is included in the prelude.

[`Requester`]: https://docs.rs/teloxide/latest/teloxide/requests/trait.Requester.html
[`Bot`]: https://docs.rs/teloxide/latest/teloxide/struct.Bot.html
[adaptors]: https://docs.rs/teloxide/latest/teloxide/adaptors/index.html

```rust,no_run
use teloxide::prelude::{*, ChatId};

# #[tokio::main]
# async fn main() -> Result<(), Box<dyn std::error::Error>> {
#     let bot = Bot::new("TOKEN").auto_send();
# 
#     // replace 0 with your user id
#     let your_id = ChatId(0);
bot
    .send_message(your_id, "Hi!")
    .await?;   
#  
#     Ok(())
# }
```

`send_message` has optional parameters that can be provided like this:

```rust,no_run
use teloxide::prelude::*;

# #[tokio::main]
# async fn main() -> Result<(), Box<dyn std::error::Error>> {
#     let bot = Bot::new("TOKEN").auto_send();
# 
#     // replace 0 with your user id
#     let your_id = ChatId(0);
bot
    .send_message(your_id, "Hi!")
    .protect_content(true) // <-- optional parameter!
    .await?;   
#  
#     Ok(())
# }
```

Normally you'd also need to call `.send()` before `.await`, but `.auto_send()` on the bot allows you to not do that (more on that later).

## How it actually works

There are three parts to Telegram methods in teloxide: payloads, requests and requesters.

### Payloads

Requests parameters are stored in plain structures called payloads, for example:

```rust,no_run
// teloxide::payloads
pub struct SendMessage {
    pub chat_id: ChatId,
    pub text: String,
    pub parse_mode: Option<ParseMode>, // optional parameters use `Option<_>`
    pub entities: Option<Vec<MessageEntity>>,
    pub disable_web_page_preview: Option<bool>,
    pub disable_notification: Option<bool>,
    pub protect_content: Option<bool>,
    pub reply_to_message_id: Option<i32>,
    pub allow_sending_without_reply: Option<bool>,
    pub reply_markup: Option<ReplyMarkup>,
}
```

Such structures implement [`Payload`] trait that specifies the name of the method and the return type:

[`Payload`]: https://docs.rs/teloxide/latest/teloxide/requests/trait.Payload.html

```rust,no_run
impl Payload for SendMessage {
    type Output = Message;
    const NAME: &'static str = "SendMessage"
}
```

For all payloads there is also a `<Payload>Setters` trait that is implemented for any `HasPayload<Payload = <Payload>>`.
`*Setters` traits add builder-functions that allow to change the payload (like `protect_content` in the example above).
All setter traits are reexported "`as _`" in the prelude, so you don't have to import traits every time you want to use an optional parameter.

[`HasPayload`] is a trait that just allows to get a `&` or `&mut` reference to a payload.
[`HasPayload`] is implemented for all payloads and simply returns `self`.

[`HasPayload`]: https://docs.rs/teloxide/latest/teloxide/requests/trait.HasPayload.html

### Requests

Requests are types that hold payloads + the information needed to make a telegram request (e.g. bot token).
Such types implement [`Request`] trait that looks roughly like this:

[`Request`]: https://docs.rs/teloxide/latest/teloxide/requests/trait.Request.html

```rust,no_run
pub trait Request: HasPayload {
    type Err: Error + Send;
    
    async fn send(self) -> Result<Output<Self>, Self::Err>;

    // some less important items are left out
}

pub type Output<T> = <<T as HasPayload>::Payload as Payload>::Output;
```

- [`Request`]s also implement [`HasPayload`], so setters work on them too
- `Err` is the type of the error that is returned when the request fails
  - It's normally [`RequestError`]
- `send` sends the request and returns the result

[`RequestError`]: https://docs.rs/teloxide/latest/teloxide/enum.RequestError.html

### Requester

The final piece in this puzzle is the [`Requester`] trait that looks roughly like this:

[`Requester`]: https://docs.rs/teloxide/latest/teloxide/prelude/trait.Requester.html

```rust,no_run
pub trait Requester {
    type Err: std::error::Error + Send;

    type SendMessage: Request<Payload = SendMessage, Err = Self::Err>;

    fn send_message<C, T>(&self, chat_id: C, text: T) -> Self::SendMessage
    where
        C: Into<ChatId>,
        T: Into<String>;

    // ~90 other methods
}
```

- `send_message` (and all other methods) return types implementing [`Request`] with a matching payload
- All requests return the same error type (`Err`)
- Some parameter types are auto-converted with `Into<>`
  - This allows to pass `i64` as `ChatId` or `&str` as `String`
  - Some parameters are also converted via `IntoIterator`
- [`Requester`] is lazy and only returns builders, but doesn't send requests eagerly

And that's all you need to know about how requests work in telegram!
The system is somewhat complicated and convoluted, but in the end it allows for a nice syntax of `bot.method(...).optional(...).send().await?`.
