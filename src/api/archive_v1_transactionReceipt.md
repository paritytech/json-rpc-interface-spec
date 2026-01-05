# archive_v1_transactionReceipt

**Parameters**:

- `transactionHash`: String containing a hexadecimal-encoded hash of the transaction to retrieve the receipt for.

**Return value**: If a transaction with that hash is found and included in a block, a `TransactionReceipt` object. If no transaction with that hash is found or it's not yet included in a block, `null`.

## TransactionReceipt Structure

If the transaction exists, the return value is a JSON object with the following structure:

```json
{
    "blockHash": "0x...",
    "blockNumber": "...",
    "transactionIndex": ...,
    "transactionHash": "0x...",
    "from": "0x...",
    "to": "0x...",
    "status": "success" | "failed",
    "events": [
        {
            "phase": "ApplyExtrinsic" | "Finalization" | "Initialization",
            "pallet": "...",
            "event": "...",
            "data": "0x..."
        }
    ],
    "fee": "0x...",
    "gasUsed": "0x...",
    "storageUsed": "0x...",
    "logs": ["0x..."],
    "executionMetadata": {
        "weightUsed": ...,
        "paysFee": "yes" | "no",
        "class": "normal" | "operational" | "mandatory"
    }
}

```
