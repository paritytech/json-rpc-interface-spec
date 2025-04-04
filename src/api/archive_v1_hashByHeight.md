# archive_v1_hashByHeight

**Parameters**:

- `height`: Integer representing the height of the block.

**Return value**: Array (possibly empty) of strings containing a hexadecimal-encoded hash of a block header.

The JSON-RPC client must find the blocks (zero, one, or more) whose height is the one passed as parameter. If the `height` is inferior or equal to the finalized block height, then only finalized blocks must be fetched and returned.

It returns an array of hexadecimal-encoded hashes corresponding to the blocks of this height that are known to the node.
If the `height` is inferior or equal to the finalized block height, the array must contain exactly one entry. Furthermore, calling this function multiple times with the same `height` inferior or equal to the finalized block height must always return the same result.
