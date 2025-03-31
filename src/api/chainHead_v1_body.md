# chainHead_v1_body

**Parameters**:

- `followSubscription`: An opaque string that was returned by `chainHead_v1_follow`.
- `hash`: String containing a hexadecimal-encoded hash of the header of the block whose body will be retrieved.

**Return value**: A JSON object.

The JSON object returned by this function has one the following formats:

### Started

```
{
    "result": "started",
    "operationId": ...
}
```

This return value indicates that the request has successfully started.

`operationId` is a string containing an opaque value representing the operation.

### LimitReached

```
{
    "result": "limitReached"
}
```

This return value indicates the request couldn't be started because the server is overloaded, or that the `followSubscription` is invalid or stale.

The JSON-RPC client should try again after an on-going `chainHead_v1_storage`, `chainHead_v1_body`, or `chainHead_v1_call` operation finishes.

The JSON-RPC server must accept at least 16 concurrent operations for any given `chainHead_v1_follow` subscription. In other words, as long as the JSON-RPC client makes sure that no more than 16 operations are in progress at any given item, it is guaranteed that all of its operations will be accepted by the JSON-RPC server.
For this purpose, each item requested through `chainHead_v1_storage` counts as one operation, and each call to `chainHead_v1_body` and `chainHead_v1_call` counts as one operation.

## Overview

The JSON-RPC server must start obtaining the body (in other words the list of transactions) of the given block.

The progress of the operation is indicated through `operationBodyDone`, `operationInaccessible`, or `operationError` notifications generated on the corresponding `chainHead_v1_follow` subscription.

The operation continues even if the target block is unpinned with `chainHead_v1_unpin`.

This function should be seen as a complement to `chainHead_v1_follow`, allowing the JSON-RPC client to retrieve more information about a block that has been reported. Use `archive_v1_body` if instead you want to retrieve the body of an arbitrary block.

## Possible errors

- If the networking part of the behaviour fails, then a `{"event": "operationInaccessible"}` notification is generated (as explained above).
- If the `followSubscription` is invalid or stale, then `"result": "limitReached"` is returned (as explained above).
- A JSON-RPC error with error code `-32801` is generated if the block hash passed as parameter doesn't correspond to any block that has been reported by `chainHead_v1_follow`, or the block hash has been unpinned.
- A JSON-RPC error with error code `-32602` is generated if one of the parameters doesn't correspond to the expected type (similarly to a missing parameter or an invalid parameter type).
- A JSON-RPC error with error code `-32603` is generated if the JSON-RPC server cannot interpret the block (hardware issues, corrupted database, disk failure etc).
