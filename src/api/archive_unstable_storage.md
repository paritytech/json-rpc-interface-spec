# archive_unstable_storage

**Parameters**:

- `hash`: String containing a hexadecimal-encoded hash of the header of the block whose storage to fetch.
- `items`: Array of objects. The structure of these objects is found below.
- `childTrie`: `null` for main storage look-ups, or a string containing the hexadecimal-encoded key of the child trie of the "default" namespace.

Each element in `items` must be an object containing the following fields:

- `key`: String containing the hexadecimal-encoded key to fetch in the storage.
- `type`: String equal to one of: `value`, `hash`, `closestDescendantMerkleValue`, `descendantsValues`, `descendantsHashes`.
- `paginationStartKey`: This parameter is optional and should be a string containing the hexadecimal-encoded key from which the storage iteration should resume. This parameter is only valid in the context of `descendantsValues` and `descendantsHashes`.

**Return value**: String containing an opaque value representing the operation.

## Notifications format

This function will later generate one or more notifications in the following format:

```json
{
    "jsonrpc": "2.0",
    "method": "archive_unstable_storageDiffEvent",
    "params": {
        "subscription": "...",
        "result": ...
    }
}
```

Where `subscription` is the value returned by this function, and `result` can be one of:

### storage

```json
{
    "event": "storage",
    "key": "0x...",
    "value": "0x...",
    "hash": "0x...",
    "closestDescendantMerkleValue": "0x...",
    "childTrieKey": "0x...",
}
```

The notification corresponds to a storage response for one of the requested items.

- `key`: String containing the hexadecimal-encoded key of the storage entry. This is guaranteed to be equal to one of the `key`s provided for `type` equal to `value`, `hash` or `closestDescendantMerkleValue`. For queries of type `descendantsValues` or `descendantsHashes`, `key` is guaranteed to start with one of the `key`s provided.

- `value` (optional): String containing the hexadecimal-encoded value of the storage entry. This field is present when the `type` of the query was `"value"` or `"descendantsValues"`.

- `hash` (optional): String containing the hexadecimal-encoded hash of the storage entry. This field is present when the `type` of the query was `"hash"` or `"descendantsHashes"`.

- `closestDescendantMerkleValue` (optional): String containing the hexadecimal-encoded Merkle value of the closest descendant of `key` (including branch nodes). This field is present when the `type` of the query was `"closestDescendantMerkleValue"`. The trie node whose Merkle value is indicated in `closestDescendantMerkleValue` is not indicated, as determining the key of this node might incur an overhead for the JSON-RPC server. The Merkle value is equal to either the node value or the hash of the node value, as defined in the [Polkadot specification](https://spec.polkadot.network/chap-state#defn-merkle-value).

- `childTrieKey` (optional): String containing the hexadecimal-encoded key of the child trie of the "default" namespace if the storage entry is part of a child trie. If the storage entry is part of the main trie, this field is not present.

### storageDone

```json
{
    "event": "storageDone",
}
```

This event is always generated after all `storage` events have been generated.

No more events will be generated after a `storageDone` event.

### storageError

```json
{
    "event": "storageError",
    "error": "...",
}
```

`error` is a human-readable error message indicating why the call has failed. This string isn't meant to be shown to end users, but is for developers to understand the problem.

No more events will be generated after a `storageError` event.


## Overview

For each item in `items`, the JSON-RPC server must start obtaining the value of the entry with the given `key` from the storage, either from the main trie or from `childTrie`. If `type` is `descendantsValues` or `descendantsHashes`, then it must also obtain the values of all the descendants of the entry.

For the purpose of storage requests, the trie root hash of the child tries of the storage can be found in the main trie at keys starting the bytes of the ASCII string `:child_storage:`. This behaviour is consistent with all the other storage-request-alike mechanisms of Polkadot and Substrate-based chains, such as host functions or libp2p network requests.

If the height of the block hash provided is less than or equal to the current finalized block height (which can be obtained via archive_unstable_finalizedHeight), then calling this method with the same parameters will always return the same response.
If the height of the block hash provided is greater than the current finalized block height, then the block might be pruned at any time and calling this method may return null.

This function should be used when the target block is older than the blocks reported by `chainHead_v1_follow`.
Use `chainHead_v1_storage` if instead you want to retrieve the storage of a block obtained by the `chainHead_v1_follow`.

If `items` contains multiple identical or overlapping queries, the JSON-RPC server can choose whether to merge or not the items in the result. For example, if the request contains two items with the same key, one with `hash` and one with `value`, the JSON-RPC server can choose whether to generate two `item` objects, one with the value and one with the hash, or only a single `item` object with both `hash` and `value` set.

It is allowed (but discouraged) for the JSON-RPC server to provide the same information multiple times in the result, for example providing the `value` field of the same `key` twice. Forcing the JSON-RPC server to de-duplicate items in the result might lead to unnecessary overhead.

## Possible errors

- A JSON-RPC error can be generated if the JSON-RPC client has to many active calls to `archive_unstable_storageDiff`.
- A JSON-RPC error with error code `-32602` is generated if one of the parameters doesn't correspond to the expected type (similarly to a missing parameter or an invalid parameter type).
