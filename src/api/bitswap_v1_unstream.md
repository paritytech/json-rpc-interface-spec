# bitswap_v1_unstream

**Parameters**:

- `subscription`: An opaque string that was returned by [`bitswap_v1_stream`](bitswap_v1_stream.md).

**Return value**: *null*

Stops a subscription previously started with [`bitswap_v1_stream`](bitswap_v1_stream.md). The
implementation cancels any remaining work for the cancelled subscription and stops emitting events.

Has no effect if the `subscription` parameter does not correspond to any active subscription started
with `bitswap_v1_stream`. No error is raised for an unknown or already-completed subscription, since
`bitswap_v1_stream` auto-completes after one event per input CID and a client may legitimately call
`bitswap_v1_unstream` after the stream has finished.

Note that there is no guarantee that the JSON-RPC client will not receive notifications between the
moment it calls `bitswap_v1_unstream` and the moment it receives the response to that call.
