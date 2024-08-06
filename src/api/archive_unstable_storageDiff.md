# archive_unstable_storageDiff

**Parameters**:

- `hash`: String containing a hexadecimal-encoded hash of the header of the block whose storage difference will be retrieved.

- `previousHash` (optional): String containing a hexadecimal-encoded hash of the header of the block from which the storage difference will be calculated. The `previousHash` must be an ancestor of the provided `hash`.  When this parameter is provided, the storage difference is calculated between the `hash` block and the `previousHash` block. If this parameter is not provided, the storage difference is calculated between the `hash` block and the parent of the `hash` block.

- `items` (optional): Array of objects. The structure of these objects is found below.

    ```json
    "items": [
        {
            "prefixes": [
                {
                    "key": "0x...",
                    "type": "value" | "hash" | "none",
                },
            ],

            "trieType": "mainTrie" | "childTrie",
            "childTrieKey": "0x...",
        },
    ]
    ```

  If the `items` parameter is not provided, the storage difference is calculated for all keys of the main storage trie.

  Each element in `items` must be an object containing the following fields:

  - `prefixes` (optional): Array of JSON objects describing how the storage difference will be calculated. Each object contains the following fields:
    - `key`: String containing a hexadecimal-encoded key prefix. Only the storage entries whose key starts with the provided prefix are returned.
    - `type`: String equal to one of: `value`, `hash`, `none`.

  - `trieType`: A string indicating the trie type for which the storage difference will be calculated. The possible values are:
    - `mainTrie`: The storage difference is calculated for the main storage.
    - `childTrie`: The storage difference is calculated for the child trie of the "default" namespace. If this type is provided, the `childTrieKey` field must be provided.

  - `childTrieKey`: String containing the hexadecimal-encoded key of the child trie of the "default" namespace. This field is only present when the `trieType` field is set to `childTrie`.

**Return value**: A JSON object.

The JSON object returned by this function has the following format:

```json
{
    "result": [
        {
            "differences": [
                {
                    "key": "0x...",
                    "value": "0x...",
                    "hash": "0x...",
                    "type": "added" | "modified" | "deleted",
                },
            ],

            "trieType": "mainTrie" | "childTrie",
            "childTrieKey": "0x...",
        }
    ],
}
```

The `result` field contains an array of objects, each containing a JSON object:

The `differences` field is an array of objects describing storage differences. Each element contains the following fields:

- `key`: String containing the hexadecimal-encoded key of the storage entry. If the key prefix was provided in the `items` parameter, the `key` field is guaranteed to start with one of the key prefixes provided.

- `value`: String containing the hexadecimal-encoded value of the storage entry. This field is present when prefixes are not provided or when the `type` field in the `items` parameter is set to `value`.

- `hash`: String containing the hexadecimal-encoded hash of the storage entry. This field is only present when the `type` field in the `items` parameter is set to `hash`.

- `type`: String indicating the type of the storage difference. The possible values are:
  - `added`: The storage entry was added.
  - `modified`: The storage entry was modified.
  - `deleted`: The storage entry was deleted.

The `trieType` field is a string indicating the trie type for which the storage difference was calculated. The possible values are `mainTrie` and `childTrie`. These have the same meaning as in the `items` parameter.

The `childTrieKey` field is a string containing the hexadecimal-encoded key of the child trie of the "default" namespace. This field is only present when the `trieType` field is set to `childTrie`.

## Overview

This function calculates the storage difference between two blocks. The storage difference is calculated by comparing the storage of the `previousHash` block with the storage of the `hash` block. If the `previousHash` parameter is not provided, the storage difference is calculated between the parent of the `hash` block and the `hash` block.

The JSON-RPC server is encouraged to accept at least one `archive_unstable_storageDiff` subscription per JSON-RPC client. Trying to open more might lead to a JSON-RPC error when calling `archive_unstable_storageDiff`. The JSON-RPC server must return an error code if the server is overloaded and cannot accept new subscriptions.

Users that want to obtain the storage difference between two blocks should use this function instead of calling `archive_unstable_storage` for each block and comparing the results.
When users are interested in the main trie storage differences, as well as in a child storage difference, they can call this function with `items: [ { "trieType": "mainTrie" }, { "trieType": "childTrie", "childTrieKey": "0x..." } ]`.

## Possible errors

- A JSON-RPC error can be generated if the JSON-RPC client has to many active subscriptions to `archive_unstable_storageDiff`.
- A JSON-RPC error with error code `-32602` is generated if one of the parameters doesn't correspond to the expected type (similarly to a missing parameter or an invalid parameter type).
