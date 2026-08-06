# statement_unstable_submit

**Parameters**:

- `encoded`: String containing the hexadecimal-encoded SCALE-encoded statement to submit.

**Return value**: An object indicating the outcome of the submission.

This function submits a statement to the statement store. If the statement is accepted, it becomes eligible to be reported later through `statement_unstable_subscribe` notifications, and the JSON-RPC server can propagate it over the peer-to-peer network.

The return value always contains a `"status"` field. Depending on the status, additional fields might be present.

Expected outcomes caused by the submitted statement are returned in the result object described below. Malformed inputs and internal failures of the JSON-RPC server instead generate JSON-RPC errors.

### new

```json
{
  "status": "new"
}
```

The statement was accepted and wasn't already present in the store.

### known

```json
{
  "status": "known"
}
```

The statement was already present in the store.

This outcome must only be used if the statement is currently present in the store. It must not be used if the statement is only known because of expired local state kept internally by the implementation.

### rejected

The statement was rejected because the store couldn't accept it. The `"reason"` field indicates why.

#### dataTooLarge

```json
{
    "status": "rejected",
    "reason": "dataTooLarge",
    "submittedSize": ...,
    "availableSize": ...
}
```

The statement data exceeds the available size for the account. `submittedSize` is the size of the submitted statement data in bytes. `availableSize` is the remaining data allowance in bytes.

#### channelPriorityTooLow

```json
{
    "status": "rejected",
    "reason": "channelPriorityTooLow",
    "submittedExpiry": ...,
    "minExpiry": ...
}
```

When submitting a statement on a channel that already has a message, the new statement must have a higher expiry than the existing one. `submittedExpiry` is the expiry of the submitted statement. `minExpiry` is the expiry of the existing channel message.

#### accountFull

```json
{
    "status": "rejected",
    "reason": "accountFull",
    "submittedExpiry": ...,
    "minExpiry": ...
}
```

The account has reached its statement limit and the submitted statement's expiry is too low to evict an existing one. `submittedExpiry` is the expiry of the submitted statement. `minExpiry` is the lowest expiry among the account's existing statements.

#### storeFull

```json
{
  "status": "rejected",
  "reason": "storeFull"
}
```

The global statement store is full and can't accept new statements.

#### noAllowance

```json
{
  "status": "rejected",
  "reason": "noAllowance"
}
```

The account has no statement allowance set on-chain.

### invalid

The statement failed validation. The `"reason"` field indicates why.

#### noProof

```json
{
  "status": "invalid",
  "reason": "noProof"
}
```

The statement has no authenticity proof.

#### badProof

```json
{
  "status": "invalid",
  "reason": "badProof"
}
```

The statement's proof failed verification.

#### encodingTooLarge

```json
{
    "status": "invalid",
    "reason": "encodingTooLarge",
    "submittedSize": ...,
    "maxSize": ...
}
```

The encoded statement exceeds the maximum allowed size. `submittedSize` is the size of the submitted encoding in bytes. `maxSize` is the maximum allowed size in bytes.

#### alreadyExpired

```json
{
  "status": "invalid",
  "reason": "alreadyExpired"
}
```

The statement's expiry field is in the past.

## Possible errors

- A JSON-RPC error with error code `-32602` is generated if the `encoded` parameter is not a valid hexadecimal string or if the bytes don't decode into a statement.
- A JSON-RPC error with error code `-32603` is generated if the JSON-RPC server encounters an unexpected internal failure while processing the statement.
