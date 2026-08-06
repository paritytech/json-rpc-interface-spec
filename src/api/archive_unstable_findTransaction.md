# archive_unstable_findTransaction

**Parameters**:

- `transactionBytes`: String containing the hexadecimal-encoded SCALE-encoded bytes of the transaction to locate. The raw bytes are used rather than the transaction hash because transaction hashes are not globally unique.
- `from`: Optional non-negative integer block height from which to begin searching, inclusive. If omitted, the search starts at the oldest block retained by the server.
- `count`: Optional positive integer giving the maximum number of results to return. If omitted, a server-defined default applies. Servers must enforce a hard upper bound on this value regardless of the value requested by the caller.

**Return value**: An array of objects, possibly empty, ordered by ascending block height and then by ascending `transactionIndex` within the same block. Each object has the following structure:

```json
{
    "blockHash": "0x...",
    "transactionIndex": 0,
    "status": "finalized"
}
```

- `blockHash`: String containing the hexadecimal-encoded hash of a block whose body contains the bytes passed as `transactionBytes` at position `transactionIndex`.
- `transactionIndex`: Non-negative integer indicating the zero-based index of the extrinsic within the block body, using the same indexing as `archive_v1_body`.
- `status`: One of `"finalized"`, `"best"`, or `"fork"`, indicating the chain placement of the block at the moment the response is produced. Entries with status `"finalized"` are stable. Entries with status `"best"` or `"fork"` may change status or disappear in subsequent calls.

The same `transactionBytes` may legitimately appear in more than one block (across competing forks, after a reorg, or in distinct finalized blocks), which is why the return value is always an array.

The response contains location data only. It does not return execution outcome, fees, weight, or dispatched events. Such information is runtime-specific and can be obtained by querying `archive_v1_storage` against the returned `blockHash`.

This method is a query against the server's local database. It is not a proof of inclusion. Callers who require trustless verification must verify a finality proof or re-execute the block themselves.

## Indexing requirement

Implementing this method requires the server to maintain an index over the extrinsic bytes of every block it retains. Substrate nodes do not maintain such an index by default. A server that does not maintain this index must not include `archive_unstable_findTransaction` in the list returned by `rpc_methods`.

## Pagination

When a response is truncated by the server's result cap, callers can continue by repeating the call with `from` set just past the height of the last returned `blockHash`.

## Possible errors

- A JSON-RPC error with error code `-32602` is generated if any parameter is of an incorrect type, or if `transactionBytes` is not valid hexadecimal.
- A JSON-RPC error may be generated if the server has too many concurrent `archive`-prefixed calls in flight.
