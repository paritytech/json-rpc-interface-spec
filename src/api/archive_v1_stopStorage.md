# archive_v1_stopStorage

**Parameters**:

- An opaque string that was returned by `archive_v1_storage`.

**Return value**: *null*

Stops a subscription started with `archive_v1_storage`. Has no effect if the opaque string is invalid or refers to a subscription that has already emitted a `{"event": "storageDone"}` event.

JSON-RPC client implementations must be aware that, due to the asynchronous nature of JSON-RPC client <-> server communication, they might still receive chain updates notifications, for example because these notifications were already in the process of being sent back by the JSON-RPC server.
