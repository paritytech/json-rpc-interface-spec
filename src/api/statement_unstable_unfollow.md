# statement_unstable_unfollow

**Parameters**:

- `followSubscription`: An opaque string that was returned by `statement_unstable_follow`.

**Return value**: _null_

Stops a follow started with `statement_unstable_follow`. Has no effect if the `followSubscription` is invalid or refers to a follow that has already emitted a `{"event": "stop"}` event.

Removing a follow automatically removes all the filters that were attached to it.

JSON-RPC client implementations must be aware that, due to the asynchronous nature of JSON-RPC client <-> server communication, they might still receive `statement_unstable_followEvent` notifications, for example because these notifications were already in the process of being sent back by the JSON-RPC server.
