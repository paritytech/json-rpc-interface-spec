# archive_v1_body

**Parameters**:

- `hash`: String containing a hexadecimal-encoded hash of the header of the block whose body will be retrieved.

**Return value**: If a block with that hash is found, an array of strings containing the hexadecimal-encoded SCALE-codec-encoded transactions in that block. If no block with that hash is found, `null`.

If the block was previously returned by `archive_v1_hashByHeight` at a height inferior or equal to the current finalized block height (as indicated by `archive_v1_finalizedHeight`), then calling this method multiple times is guaranteed to always return non-null and always the same result.

If the block was previously returned by `archive_v1_hashByHeight` at a height strictly superior to the current finalized block height (as indicated by `archive_v1_finalizedHeight`), then the block might "disappear" and calling this function might return `null` at any point.
