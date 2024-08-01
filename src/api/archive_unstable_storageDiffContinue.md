# archive_unstable_storageDiffContinue

**Parameters**:

- `subscription`: An opaque string that was returned by `archive_unstable_storageDiff`.

**Return value**: *null*

Continues a subscription started with `archive_unstable_storageDiff`. Has no effect if the `subscription` is invalid or refers to a subscription that has already emitted a `{"event": "done"}` event.

## Possible errors

- A JSON-RPC error generated if the `subscription` is valid but haven't generated a `{"event": "continue"}` event.
