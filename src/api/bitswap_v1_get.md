# bitswap_v1_get

**Parameters**:

- `cid`: CID of the data chunk requested serialized in a [string format](https://github.com/multiformats/cid/blob/edb1c5294ad2d8257812d7ded4941c3e0fafccf3/README.md#variant---stringified-form).  
  Only CIDv1 version in `base32` multibase encoding (string starting from `b...`) is supported.  
  Only `sha2-256` and `blake2b-256` hash functions are supported.  
  Example: `bafk2bzacec5lindttapqst35gv4767ig2vxmcaeherlaubtqhufg4tqqmeen4`.


**Return value**: String containing the requested data chunk, hexadecimal-encoded. Always starts
with `0x...`.  
Because Bitswap chunks are limited by 2 MiB at the transport layer, the maximum returned string size
is 4 MiB + 2 B.

**Errors**: If the data chunk retrieval fails, JSON-RPC call fails. The error code
indicates a possible retry strategy.

## Error categories

| Code   | Category            | Meaning                                                 |
| ------ | ------------------- | ------------------------------------------------------- |
| -32602 | `InvalidParams`     | Invalid/unsupported CID was passed                      |
| -32810 | `Fail`              | Permanent failure for this request, e.g. data not found |
| -32811 | `FailRetry`         | Transient failure, can retry immediately                |
| -32812 | `FailRetryBackoff`  | Transient failure, can retry with a delay               |

`InvalidParams` is a standard JSON-RPC error indicating invalid data passed to the method.
The client must not call the method with the same argument again.

For error category `Fail` retrying doesn't make sense unless the data has been added to the network
after the failure.

Error category `FailRetry` can be retried immediately. For example, request timeout falls into this
category. Even though the client can retry immediately, the implementations are encouraged to
rate-limit the retry attempts and limit the total number of retries.

Error category `FailRetryBackoff` can be retried after a delay. For example, such error can be
generated if no peers are currently connected to the light client. Recommended delay before retrying
is 1-5 seconds.

Detailed error information for debugging/logging purposes can be obtained from the JSON-RPC error
response. This information is implementation-specific and subject to change, so must not be used
programmatically.

### Detailed error information

| Field          | Description                          | Example value                         |
| -------------- | ------------------------------------ | ------------------------------------- |
| `message`      | Human-readable error description     | `Request timeout.`                    |
| `data.variant` | Error variant for structured logging | `Timeout`, `NoPeers`, `NotFound`, ... |

### Example JSON-RPC `error` field

```json
{"code": -32812, "message": "No Bitswap peers connected", "data": {"variant": "NoPeers"}}
```

Please note again that only the `code` and corresponding error retry categories in the table above
are stable. Everything else is provided for debugging purposes only, subject to change, and must not
be relied upon in business logic.
