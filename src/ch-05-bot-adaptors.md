# Bot adaptors

Bot adaptors are types that wrap other types that implement [`Requester`] trait and implement [`Requester`] trait themselves.
This allows to easily add opt-in requester behaviour.

[`Requester`]: https://docs.rs/teloxide/latest/teloxide/prelude/trait.Requester.html

In this chapter we'll see what adaptors does teloxide provide.

##  `AutoSend`

This is probably the most useful adaptor. 
Its request types implement `Future`, which allows to use `.await` directly without using `.send()`.

> Note: `AutoSend` must be the outermost adaptor, or you won't be able to `.await` requests directly

##  `DefaultParseMode`

Allows specifying a default parse mode that will be used for all methods that support parse mode.

## `CacheMe`

Bots user object rarely changes, so we can cache it.

## `Trace`

Logs every request, useful for debugging.

## `ErasedRequester`

That one is a little different from other adapters.
While most adaptors have a generic that represents what they are wrapping, `ErasedRequester` doesn't.
`ErasedRequester` can be created from any type that implements `Request` and doesn't change behaviour.

It is useful if you need to store bots with different types in the same variable/array/etc.
See this example: [`core/examples/erased.rs`].

[`core/examples/erased.rs`]: https://github.com/teloxide/teloxide-core/blob/d0be26057512764d00b9ed3e3d2237b625b5ae91/examples/erased.rs#L32-L36
