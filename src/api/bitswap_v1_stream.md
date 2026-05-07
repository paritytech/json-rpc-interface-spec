# bitswap_v1_stream

**Parameters**:

- `cids`: Array of CID strings, with the same constraints as the `cids` parameter of [`bitswap_v1_getMany`](bitswap_v1_getMany.md).  
  The implementation-defined maximum applies.

**Return value**: String representing the subscription.

The string returned by this function is opaque and its meaning can't be interpreted by the JSON-RPC
client. It is only meant to be matched with the `subscription` field of events and potentially passed
to `bitswap_v1_unstream`.

`bitswap_v1_stream` and [`bitswap_v1_getMany`](bitswap_v1_getMany.md) are equivalent in the data they
deliver: both produce exactly one outcome per input CID. They differ in delivery: `bitswap_v1_getMany`
returns a single ordered array once every CID has resolved, whereas `bitswap_v1_stream` emits each
outcome as soon as it is ready, in the order results become available — **not** in input order.
This lets the client begin processing the first chunks while later chunks are still in flight.
The client correlates each event with its request via the `cid` field embedded in `result`.

If `bitswap_v1_unstream` is called or the JSON-RPC client disconnects, the implementation cancels
remaining work and stops emitting events.

## Notifications format

This function will generate exactly one notification per input CID, in arrival order (the order in
which each CID resolves), in the following format:

```json
{
    "jsonrpc": "2.0",
    "method": "bitswap_v1_streamEvent",
    "params": {
        "subscription": "...",
        "result": ["<cid>", <BlockResult>]
    }
}
```

Where `subscription` is the value returned by this function, and `result` is a 2-element array
`[cid, BlockResult]`. The shape of `BlockResult` is identical to
[`bitswap_v1_getMany`](bitswap_v1_getMany.md):

**Ok**:

```json
[ "<cid>", "0x..." ]
```

**Err**:

```json
[ "<cid>", { "code": -32810, "message": "..." } ]
```

`code` carries the same [error categories](bitswap_v1_get.md#error-categories) as the top-level error
of [`bitswap_v1_get`](bitswap_v1_get.md). Clients that already know how to interpret `code` for
`bitswap_v1_get` can reuse the same retry logic per-CID.

`message` is a human-readable diagnostic string for logs and developer-facing tooling. Only `code` is
stable for programmatic dispatch.

A missing or invalid CID in the input produces an `Err` event for that CID and does not abort the
rest of the stream. Because events are emitted in arrival order, an `Err` event for one CID may be
emitted before, after, or interleaved with `Ok` events for other CIDs.

## End of stream

After exactly one event per input CID has been emitted, the implementation has no further work to do.
JSON-RPC subscriptions do not have a standard "end-of-stream" marker; the client knows the stream is
exhausted because it has received the same number of events as input CIDs.

The client may then call `bitswap_v1_unstream` to release the subscription, or simply stop reading.

## Possible errors

Whole-call failures cause the subscription to be rejected (no events are emitted). See [error categories](bitswap_v1_get.md#error-categories).

## Empty input

An empty input array (`cids: []`) is allowed. The subscription opens, no events are emitted, and the
client can release it via `bitswap_v1_unstream` whenever convenient.

## Duplicate input

The implementation rejects subscriptions whose input contains the same CID more than once. Detection
is two-stage:

1. Two literally-identical input strings (regardless of whether they are valid CIDs) → rejected.
2. Two cosmetically different CID strings that decode to the same 32-byte content digest →
   rejected.

In either case the subscription is rejected at the top level with `-32602 InvalidParams` (no events
are emitted). Callers must deduplicate input before subscribing.

Duplicates are rejected so that `bitswap_v1_stream` can guarantee exactly one event per input CID —
the client uses the input array length to determine when the stream is exhausted. Without this
guarantee, the input length would not be a reliable end-of-stream signal.
