# archive_unstable_storageDiff

**Parameters**:

- `hash`: String containing a hexadecimal-encoded hash of the header of the block whose storage difference will be retrieved.

- `previousHash` (optional): String containing a hexadecimal-encoded hash of the header of the block from which the storage difference will be calculated. When this parameter is provided, the storage difference is calculated between the `hash` block and the `previousHash` block. If this parameter is not provided, the storage difference is calculated between the `hash` block and the parent of the `hash` block.

- `items` (optional): JSON object containing the following fields:

    ```json
    "items": {
        "prefixes": [
            {
                "key": "0x...",
                "type": "value" | "hash" | "none",
            },
        ],
        "childTrie": "0x...",
    }
    ```

  If the `items` parameter is not provided, the storage difference is calculated for all keys of the main storage trie.
  - `prefixes` (optional): Array of JSON objects describing how the storage difference will be calculated. Each object contains the following fields:
    - `key`: String containing the hexadecimal-encoded key prefix under which the storage difference is calculated.
    - `type`: String equal to one of: `value`, `hash`, `none`.
  - `childTrie` (optional): A string containing the hexadecimal-encoded key of the child trie of the "default" namespace. If this parameter is not provided, the storage difference is calculated for the main storage.

**Return value**: String containing an opaque value representing the operation.

This function calculates the storage difference between two blocks. The storage difference is calculated by comparing the storage of the `previousHash` block with the storage of the `hash` block. If the `previousHash` parameter is not provided, the storage difference is calculated between the parent of the `hash` block and the `hash` block.

The storage difference is calculated for the main storage trie by default. The `items` parameter can be used to specify the key prefixes for which the storage difference will be calculated. The `prefixes` field contains an array of objects, each describing a key prefix and the type of the storage difference to calculate.

The JSON-RPC server is encouraged to accept at least one `archive_unstable_storageDiff` subscription per JSON-RPC client. Trying to open more might lead to a JSON-RPC error when calling `archive_unstable_storageDiff`. The JSON-RPC server must return an error code if the server is overloaded and cannot accept new subscriptions.

## Notifications format

This function will later generate notifications with the following format:

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

Where `subscription` is the value returned by this function, and `result` can be one of the following:

### differences

```json
{
    "event": "differences",
    "differences": [
        {
            "key": "0x...",
            "value": "0x...",
            "hash": "0x...",
            "type": "added" | "modified" | "deleted",
        },
    ],
}
```

The `differences` field contains an array of objects, each describing a storage difference.

The `key` field contains the hexadecimal-encoded key of the storage entry. If the a key prefix was provided in the `items` parameter, the `key` field is guaranteed to start with one of the key prefixes provided.

The `value` field contains the hexadecimal-encoded value of the storage entry. This field is present when prefixes are not provided or when the `type` field in the `items` parameter is set to `value`.

The `hash` field contains the hexadecimal-encoded hash of the storage entry. This field is only present when the `type` field in the `items` parameter is set to `hash`.

When the `type` field in the `items` parameter is set to `none`, the `value` and `hash` fields are not present.

The `type` field indicates the type of the storage difference. The possible values are:

- `added`: The storage entry was added.
- `modified`: The storage entry was modified.
- `deleted`: The storage entry was deleted.

### continue

```json
{
    "event": "continue",
}
```

This notification is sent when the storage difference calculation is paused and needs to be resumed. The user must call `archive_unstable_storageDiffContinue` with the subscription value to continue the calculation.

### done

```json
{
    "event": "done",
}
```

This notification is sent when the storage difference calculation is complete.

## Possible errors

- A JSON-RPC error can be generated if the JSON-RPC client has to many active subscriptions to `archive_unstable_storageDiff`.
- A JSON-RPC error with error code `-32602` is generated if one of the parameters doesn't correspond to the expected type (similarly to a missing parameter or an invalid parameter type).
