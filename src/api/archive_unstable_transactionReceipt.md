# archive_unstable_transactionReceipt

**Parameters**:

- `transactionHash`: String containing a hexadecimal-encoded hash of the transaction to locate.

**Return value**: Array of `TransactionLocation` objects, or empty array if no transaction with that hash is found.

If the transaction exists in one or more blocks, returns an array of JSON objects with the following structure:

```json
[
    {
        "blockHash": "0x...",
        "blockNumber": "...",
        "transactionIndex": ...,
        "encodedTransaction": "0x..."
    }
]
```
