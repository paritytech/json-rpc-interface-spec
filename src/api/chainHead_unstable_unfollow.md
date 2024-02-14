# chainHead_unstable_unfollow

**Parameters**:

- `followSubscription`: An opaque string that was returned by `chainHead_unstable_follow`.

**Return value**: *null*

Stops a subscription started with `chainHead_unstable_follow`. Has no effect if the `followSubscription` is invalid or refers to a subscription that has already emitted a `{"event": "stop"}` event.

JSON-RPC client implementations must be aware that, due to the asynchronous nature of JSON-RPC client <-> server communication, they might still receive chain updates notifications, for example because these notifications were already in the process of being sent back by the JSON-RPC server.

This function can only be called from the same connection as the one that made the `chainHead_unstable_follow` subscription.
