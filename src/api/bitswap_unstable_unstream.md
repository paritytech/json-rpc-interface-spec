# bitswap_unstable_unstream

**Parameters**:

- `subscription`: An opaque string that was returned by [`bitswap_unstable_stream`](bitswap_unstable_stream.md).

**Return value**: *null*

Stops a subscription previously started with [`bitswap_unstable_stream`](bitswap_unstable_stream.md). The implementation cancels any remaining work for the cancelled subscription and stops emitting events.

Has no effect if the `subscription` parameter does not correspond to any active subscription started with `bitswap_unstable_stream`. No error is raised for an unknown or already-completed subscription, since `bitswap_unstable_stream` auto-completes with a `streamDone` event after every input CID has been resolved and a client may legitimately call `bitswap_unstable_unstream` after the stream has finished.

Note that there is no guarantee that the JSON-RPC client will not receive notifications between the moment it calls `bitswap_unstable_unstream` and the moment it receives the response to that call.
