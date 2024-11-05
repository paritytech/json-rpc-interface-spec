# archive_unstable_storageDiff

**Parameters**:

- `hash`: String containing a hexadecimal-encoded hash of the header of the block whose storage difference will be retrieved.

- `items`: Array of objects. The structure of these objects is found below.

- `previousHash` (optional): String containing a hexadecimal-encoded hash of the header of the block from which the storage difference will be calculated. The `previousHash` must be an ancestor of the provided `hash`.  When this parameter is provided, the storage difference is calculated between the `hash` block and the `previousHash` block. If this parameter is not provided, the storage difference is calculated between the `hash` block and the parent of the `hash` block.

    ```json
    "items": [
        {
            "key": "0x...",
            "returnType": "value" | "hash",
            "childTrieKey": "0x...",
        },
    ]
    ```

  If the `items` parameter is not provided, the storage difference is calculated for all keys of the main storage trie.

  Each element in `items` must be an object containing the following fields:

  - `key` (optional): String containing a hexadecimal-encoded key prefix. Only the storage entries whose key starts with the provided prefix are returned. If this field is not present, the storage difference is calculated for all keys of the trie.
  - `returnType`: String indicating the return type of the storage under the given `key`. The possible values are:
    - `value`: The result contains the hexadecimal-encoded value of the storage entry.
    - `hash`: The result contains the hexadecimal-encoded hash of the storage entry.
  - `childTrieKey` (optional): String containing the hexadecimal-encoded key of the child trie of the "default" namespace. If this field is not present, the storage difference is calculated for the main storage trie.

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

### storageDiff

The JSON object returned by this function has the following format:

```json
{
    "event": "storageDiff",
    "key": "0x...",
    "value": "0x...",
    "hash": "0x...",
    "type": "added" | "modified" | "deleted",
    "childTrieKey": "0x...",
}
```

The `storageDiff` event is generated for each storage difference between the two blocks.

- `key`: String containing the hexadecimal-encoded key of the storage entry. A prefix of this key may have been provided in the items input.

- `value` (optional): String containing the hexadecimal-encoded value of the storage entry. This field is present when the `returnType` field in the `items` parameter is set to `value`.

- `hash` (optional): String containing the hexadecimal-encoded hash of the storage entry. This field is present when the `returnType` field in the `items` parameter is set to `hash`.

- `type`: String indicating the type of the storage difference. The possible values are:
  - `added`: The storage entry was added.
  - `modified`: The storage entry was modified.
  - `deleted`: The storage entry was deleted.

- `childTrieKey` (optional): String containing the hexadecimal-encoded key of the child trie of the "default" namespace if the storage entry is part of a child trie. If the storage entry is part of the main trie, this field is not present.

### storageDiffDone

The JSON object returned by this function has the following format:

```json
{
    "event": "storageDiffDone",
}
```

This event is always generated after all `storageDiff` events have been generated.

## Overview

This function calculates the storage difference between two blocks. The storage difference is calculated by comparing the storage of the `previousHash` block with the storage of the `hash` block. If the `previousHash` parameter is not provided, the storage difference is calculated between the parent of the `hash` block and the `hash` block.

The JSON-RPC server is encouraged to accept at least one `archive_unstable_storageDiff` subscription per JSON-RPC client. Trying to make more calls might lead to a JSON-RPC error when calling `archive_unstable_storageDiff`. The JSON-RPC server must return an error code if the server is overloaded and cannot accept new subscription call.

Users that want to obtain the storage difference between two blocks should use this function instead of calling `archive_unstable_storage` for each block and comparing the results.
When users are interested in the main trie storage differences, as well as in a child storage difference, they can call this function with `items: [ { "returnType": "value" }, { "returnType": "value", "childTrieKey": "0x..." } ]`.

## Possible errors

- A JSON-RPC error can be generated if the JSON-RPC client has to many active calls to `archive_unstable_storageDiff`.
- A JSON-RPC error can be generated if the `previousHash` parameter is provided, but the `previousHash` block is not an ancestor of the `hash` block.
- A JSON-RPC error can be generated if one of the hashes provided does not correspond to a known block header hash.
- A JSON-RPC error with error code `-32602` is generated if one of the parameters doesn't correspond to the expected type (similarly to a missing parameter or an invalid parameter type).
