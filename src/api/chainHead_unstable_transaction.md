# chainHead_unstable_transaction

**Parameters**:

- `followSubscription`: An opaque string that was returned by `chainHead_unstable_follow`.
- `transaction`: String containing the hexadecimal-encoded SCALE-encoded transaction to try to include in a block.

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

The JSON-RPC client should try again after an on-going `chainHead_unstable_transaction`, `chainHead_unstable_storage`, `chainHead_unstable_body`, or `chainHead_unstable_call` operation finishes.

The JSON-RPC server must accept at least 16 concurrent operations for any given `chainHead_unstable_follow` subscription. In other words, as long as the JSON-RPC client makes sure that no more than 16 operations are in progress at any given item, it is guaranteed that all of its operations will be accepted by the JSON-RPC server.
For this purpose, each item requested through `chainHead_unstable_transaction` counts as one operation, and each call to `chainHead_unstable_storage`, `chainHead_unstable_body` and `chainHead_unstable_call` counts as one operation.

## Overview

One can build a mental model in order to understand which events can be generated. While a transaction is being watched, it has the following properties:

- `isValidated`: `yes` or `not-yet`. A transaction is initially `not-yet` validated. A `validated` event indicates that the transaction has now been validated. After a certain number of blocks or in case of retractation, a transaction automatically becomes `not-yet` validated and needs to be validated again. No event is generated to indicate that a transaction is no longer validated, however a `validated` event will be generated again when a transaction is validated again.

- `numBroadcastedPeers`: _integer_. A transaction is initially broadcasted to 0 other peers. After a transaction is in the `isValidated: yes`, the number of broadcaster peers can increase. This number never decreases and is never reset to 0, even if a transaction becomes `isValidated: not-yet`. The `broadcasted` event is used to report about updates to this value.

- `blockIncluded`: A transaction is included in a block, or blocks, one of which should eventually become the `finalized` block of the chain.

JSON-RPC servers are allowed to skip sending events as long as it properly keeps the JSON-RPC client up to date with the state of the transaction.

## Possible errors

- If the networking part of the behaviour fails, then a `{"event": "operationInaccessible"}` notification is generated (as explained above).
- If the `followSubscription` is invalid or stale, then `"result": "limitReached"` is returned (as explained above).
- A JSON-RPC error is generated if the block hash passed as parameter doesn't correspond to any block that has been reported by `chainHead_unstable_follow`.
- A JSON-RPC error is generated if the `followSubscription` is valid but the block hash passed as parameter has already been unpinned.
