# chainHead_unstable_stopStorage

**Parameters**:

- `operationId`: An opaque string that was returned by `chainHead_unstable_storage`.

**Return value**: *null*

Stops a storage fetch started with `chainHead_unstable_storage`. If the storage fetch was still in progress, this interrupts it. If the storage fetch was already finished, this call has no effect.

JSON-RPC client implementations must be aware that, due to the asynchronous nature of JSON-RPC client <-> server communication, they might still receive a notification about this storage fetch, for example because this notification was already in the process of being sent back by the JSON-RPC server.
