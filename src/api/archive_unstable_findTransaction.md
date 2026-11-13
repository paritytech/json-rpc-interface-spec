# archive_unstable_findTransaction

**Parameters**:

- `transactionHash`: String containing the hexadecimal-encoded hash of the transaction to locate.
- `from`: String containing the hexadecimal-encoded hash of a block of the best chain. The search starts at the child of this block, so `from` itself is not searched.

**Return value**: `null` if the transaction is not present in any block of the best chain that descends from `from`. Otherwise, an object describing the first block of the best chain after `from` that contains the transaction:

```json
{
    "blockHash": "0x...",
    "transactionIndex": 3,
    "finalized": true
}
```

- `blockHash`: String containing the hexadecimal-encoded hash of the block whose body contains the transaction.
- `transactionIndex`: Non-negative integer indicating the zero-based index of the transaction within the block body, using the same indexing as `archive_v1_body`.
- `finalized`: Boolean indicating whether the block is finalized at the moment the response is produced. If `false`, the block might later leave the best chain.

A natural value for `from` is the latest finalized block known to the JSON-RPC client before it broadcast the transaction.

A transaction can be included in more than one block. In order to look for later inclusions, the JSON-RPC client can call this method again, passing the returned `blockHash` as `from`. If the transaction is present more than once within the same block, only the lowest `transactionIndex` is returned. The JSON-RPC client can use `archive_v1_body` on that block to find the others.

Blocks that are not part of the best chain are never searched. JSON-RPC clients interested in forks should use `chainHead` instead.

The JSON-RPC server might only be able to answer for part of the chain, for example if it only indexes the transactions of its most recent blocks. If it can't search every block of the best chain after `from`, it must return the `-32821` error rather than `null`. A `null` return value always means that the transaction is not in the best chain after `from`, and never that the JSON-RPC server doesn't know.

## Trust

A returned inclusion can be verified cheaply: once the JSON-RPC client trusts the header of `blockHash`, it can fetch the body of the block and check it against the `extrinsics_root` of that header. A `null` return value, however, can't be verified in an efficient and trustless way.

## Possible errors

- A JSON-RPC error with error code `-32602` is generated if one of the parameters doesn't correspond to the expected type (similarly to a missing parameter or an invalid parameter type).
- A JSON-RPC error with error code `-32820` is generated if the block hash passed as `from` doesn't correspond to any block known by the JSON-RPC server, or if that block isn't part of the best chain.
- A JSON-RPC error with error code `-32821` is generated if the JSON-RPC server can't search every block of the best chain after `from`.
- A JSON-RPC error can be generated if the JSON-RPC client has too many active calls to `archive`-prefixed functions.
