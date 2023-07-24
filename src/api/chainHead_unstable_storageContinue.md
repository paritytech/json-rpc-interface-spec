# chainHead_unstable_storageContinue

**Parameters**:

- `operationId`: An opaque string that was returned by `chainHead_unstable_storage`.

**Return value**: *null*

Resumes a storage fetch started with `chainHead_unstable_storage` after it has generated a `waiting-for-continue` event.

Has no effect if the `operationId` is invalid or refers to an operation that has emitted a `{"event": "inaccessible"}` event.

## Possible errors

- A JSON-RPC error is generated if the `operationId` is valid but hasn't generated a `waiting-for-continue` event.
