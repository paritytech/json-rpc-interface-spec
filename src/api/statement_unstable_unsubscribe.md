# statement_unstable_unsubscribe

**Parameters**:

- `subscription`: An opaque string that was returned by `statement_unstable_subscribe`.

**Return value**: _null_

Stops a subscription started with `statement_unstable_subscribe`. Has no effect if the `subscription` is invalid or refers to a subscription that has already emitted a `{"event": "stop"}` event.

Removing a subscription automatically removes all the filters that were attached to it.

JSON-RPC client implementations must be aware that, due to the asynchronous nature of JSON-RPC client <-> server communication, they might still receive `statement_unstable_subscribeEvent` notifications, for example because these notifications were already in the process of being sent back by the JSON-RPC server.
