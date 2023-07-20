# chainHead_unstable_body

**Parameters**:

- `followSubscription`: An opaque string that was returned by `chainHead_unstable_follow`.
- `hash`: String containing an hexadecimal-encoded hash of the header of the block whose body to fetch.

**Return value**: A JSON object.

The JSON object returned by this function has one the following formats:

### Started

```
{
    "result": "started",
    "subscriptionId": ...
}
```

This return value indicates that the request has successfully started.

`subscriptionId` is a string containing an opaque value representing the operation.

### LimitReached

```
{
    "result": "limitReached"
}
```

This return value indicates the request couldn't be started because the server is overloaded.

The JSON-RPC client should try again after an on-going [`chainHead_unstable_storage`], [`chainHead_unstable_body`], or [`chainHead_unstable_call`] operation finishes.

The JSON-RPC server must accept at least 16 concurrent operations for any given [`chainHead_unstable_follow`] subscription. In other words, as long as the JSON-RPC client makes sure that no more than 16 operations are in progress at any given item, it is guaranteed that all of its operations will be accepted by the JSON-RPC server.
For this purpose, each item requested through [`chainHead_unstable_storage`] counts as one operation, and each call to [`chainHead_unstable_body`] and [`chainHead_unstable_call`] counts as one operation.

### Disjoint

```
{
    "result": "disjoint"
}
```

This return value indicates that the provided `followSubscription` is invalid or stale.

## Overview

The JSON-RPC server must start obtaining the body (in other words the list of transactions) of the given block.

The operation will continue even if the given block is unpinned while it is in progress.

This function should be seen as a complement to `chainHead_unstable_follow`, allowing the JSON-RPC client to retrieve more information about a block that has been reported. Use `archive_unstable_body` if instead you want to retrieve the body of an arbitrary block.

## Notifications format

This function will later generate a notification in the following format:

```json
{
    "jsonrpc": "2.0",
    "method": "chainHead_unstable_bodyEvent",
    "params": {
        "subscription": "...",
        "result": ...
    }
}
```

Where `subscription` is the value returned by this function, and `result` can be one of:

### done

```json
{
    "event": "done",
    "value": [...]
}
```

The `done` event indicates that everything was successful.

`value` is an array of strings containing the hexadecimal-encoded SCALE-encoded extrinsics found in this block.

**Note**: Note that the order of extrinsics is important. Extrinsics in the chain are uniquely identified by a `(blockHash, index)` tuple.

No more event will be generated with this `subscription`.

### inaccessible

```json
{
    "event": "inaccessible"
}
```

The `inaccessible` event indicates that the body has failed to be retrieved from the network.

Trying again later might succeed.

No more event will be generated with this `subscription`.

## Possible errors

- If the networking part of the behaviour fails, then a `{"event": "inaccessible"}` notification is generated (as explained above).
- If the `followSubscription` is invalid or stale, then `"result": "disjoint"` is returned (as explained above).
- A JSON-RPC error is generated if the block hash passed as parameter doesn't correspond to any block that has been reported by `chainHead_unstable_follow`.
- A JSON-RPC error is generated if the `followSubscription` is valid but the block hash passed as parameter has already been unpinned.
