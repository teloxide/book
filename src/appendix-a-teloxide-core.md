# A - teloxide-core

As you may, or may not have noticed, teloxide consists of two crates -- `teloxide` (main crate) and `teloxide-core` (core crate).
Main crate depends on the core crate and reexports everything from it.

Core crate contains
- Telegram types (like `Update`, `Message`, `Chat`)
- Everything related to making requests (payloads, requests, requesters)
- Bot adaptors (like `AutoSend`, `ErasedRequester`)
- Errors

i.e. everything that is related to the Telegram Bot API.

The main crate on the other hand contains
- Reexports of everything from core crate
- `repl`s
- Dispatching systems (that allow you to process updates) 
- Update listeners
- Utils to work with formatting
- Utils to work with commands

i.e. the more high level stuff built on top of the Telegram Bot API.

Normally you want to use `teloxide`, but in rare cases when you don't need any high-level APIs, you may use `teloxide-core` directly.
