# statement_submit

**Parameters**:

- `encoded`: String containing the hexadecimal-encoded SCALE-encoded [statement](https://docs.rs/sp-statement-store/25.0.0/src/sp_statement_store/lib.rs.html#340).

**Return value**: An object indicating the outcome of the submission.

The return value always contains a `"status"` field. Depending on the status, additional fields may be present.

### new

```json
{
    "status": "new"
}
```

The statement was accepted and is new to the store.

### known

```json
{
    "status": "known"
}
```

The statement was already present in the store.

### knownExpired

```json
{
    "status": "knownExpired"
}
```

The statement was already known but has expired.

### rejected

The statement was rejected because the store cannot accept it. The `"reason"` field indicates why.

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

The global statement store is full and cannot accept new statements.

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

### internalError

```json
{
    "status": "internalError",
    ...
}
```

An internal store error occurred. This typically indicates a database or storage problem and is not caused by the submitted statement itself.

## Possible errors

- A JSON-RPC error with error code `-32602` is generated if the `encoded` parameter is not a valid hexadecimal string.
