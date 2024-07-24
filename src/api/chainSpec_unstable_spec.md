# chainSpec_unstable_spec

**Parameters**: *none*

**Return value**: A JSON object.

The JSON object returned by this function has the following format:

```json
{
    "id": "...",

    "bootnodes": [
        "...",
    ],

    "genesis": {
        "raw": {
            ..
        },
        "stateRootHash": "0x...",
    },

    // Optional
    "codeSubstitutes": {
        "...": "0x...",
    },

    // Optional
    "telemetryEndpoints": [
        {
            "address": "...",
            "verbosity_level": 0,
        }
    ],

    // Optional
    "networkProperties": {
        "protocolId": "...",
        "forkId": "...",
    },

    // Optional
    "parachain": {
        "id": 0,
    },

    // Optional
    "checkpoint": {
        "chainInformation": {
            ...
        },

        "trustedBlocks": [
            {
                "blockNumber": "...",
                "blockHash": "0x...",
            }
        ],

        "badBlocks": [
            "0x...",
        ],
    }
}
```

## Description

### Id

The `id` is a string containing the identifier of the chain.

### Bootnodes

The `bootnodes` is an array of strings containing the [multi adresss format](https://github.com/multiformats/multiaddr) of the bootnodes of the chain.  
For example, the `"/dns/polkadot-bootnode-0.polkadot.io/tcp/30333/p2p/12D3KooWSz8r2WyCdsfWHgPyvD8GKQdJ1UAiRmrcrs8sQB3fe2KU"` is a valid p2p multiaddr, where `12D3KooWSz8r2WyCdsfWHgPyvD8GKQdJ1UAiRmrcrs8sQB3fe2KU` represents the peer ID of the bootnode.
To establish a connection to the chain at least one bootnode is needed.  

### Genesis

The `genesis` is a JSON object containing the genesis storage of the chain. This field has the following formats, depending on the information available to the node.

#### Raw Format

```json
{
    "raw": {
        "top": {
            "0x..": "0x..",
            ...
        },
    }
}
```

This format is used when the node has access to the raw genesis storage and chooses to provide it.
The `"top"` field contains a map of keys to values for the top-level storage entries of the genesis. Both the keys and the values are hexadecimal-encoded strings.  
The provided keys are guaranteed to be unique and to not be a default storage child key. A default storage child key is a key that is prefixed with `b":child_storage:default:"`. For example, the wasm code of the runtime is stored under the key `b":code"`.

#### State Root Hash

```json
{
    "stateRootHash": "0x...",
}
```

This format is used when the node does not have access to the raw genesis storage, or chooses to provide only the state root hash.
The `"stateRootHash"` contains a hexadecimal-encoded string representing the Merkle value of the genesis block.

### Code Substitues

The  `codeSubstitutes` is an _optional_ JSON object containing a key-value map of code substitutes of the chain. The key is an unsigned integer represented as string indicating the block number at which the code substitute is applied. The value is a hexadecimal-encoded string representing the wasm runtime code, which is normally found under `b:code:` key.  
The given wasm runtime code is used to substitute the runtime code starting from the provided block number and until the spec-version of the chain changes.  
A substitute should be used when runtime upgrades cannot fix an underlying issue. For example, when the runtime upgrade panics.

### Telemetry Endpoints

The `telemetryEndpoints` is an _optional_ array of objects containing the telemetry endpoints of the chain.  
Each object has the following format:

```json
{
    "address": "...",
    "verbosity_level": 0,
}
```

- `"address"` is a string containing the address of the telemetry server. The address can be specified in the URL format, or in the multi address format.  
- `"verbosity_level"` is an unsigned integer indicating the verbosity level of the telemetry server. The verbosity ranges from 0 to 9, where 0 is the least verbose and 9 is the most verbose.

### Network Properties

The `networkProperties` is an _optional_ JSON object containing the network properties of the chain. The object has the following format:

```json
{
    "forkId": "...",
}
```

- `forkId` is an _optional_ string containing the fork id that identifies the chain. In the case where two chains have the same genesis, this field can be used to signal a fork on the networking level.

### Parachain

- `parachain` is an _optional_ JSON object containing the parachain information. This fields is present only if the client spec describes a parachain. The object has the following format:

```json
{
    "id": 0,
    "relayChain": "..."
}
```

- `id` is an unsigned integer indicating the id of the parachain.
- `relayChain` is a string containing the identifier of the relay chain.

### Checkpoint

The `checkpoint` is an _optional_ JSON object containing the checkpoint of the chain. This information could be used to synchronize to with the head of the chain.

#### ChainInformation

The `"chainInformation"` field is a JSON object. This object has the following format:

```json
{
    "finalized_block_header": "0x...",

    "consensus": {
        "consensusType": "Aura" | "Babe" | "AllAuthorized",

        "aura": {
            "authorities": [
                {
                    "publicKey": "0x...",
                },
            ],

            "slot_duration": 0,
        },

        "babe": {
            "slotPerEpoch": 0,

            "finalizedCurrentEpoch": {
                "epochIndex": 0,
                "startSlotNumber": 0,
                "authorities": [
                    {
                        "publicKey": "0x...",
                        "weight": 0,
                    },
                ],
                "randomness": "0x...",
                "constant": {
                    "numerator": 0,
                    "denominator": 0,
                },
                "allowedSlots": "PrimarySlots" | "PrimaryAndSecondaryPlainSlots" | "PrimaryAndSecondaryVRFSlots",
            },

            "finalizedNextEpoch": {
                "epochIndex": 0,
                "startSlotNumber": 0,
                "authorities": [
                    {
                        "publicKey": "0x...",
                        "weight": 0,
                    },
                ],
                "randomness": "0x...",
                "constant": {
                    "numerator": 0,
                    "denominator": 0,
                },
                "allowedSlots": "PrimarySlots" | "PrimaryAndSecondaryPlainSlots" | "PrimaryAndSecondaryVRFSlots",
            }
        },
    },

    "finality": {
        "finalityType": "Grandpa" | "Outsourced",

        "grandpa": {
            "authoritiesSetId": 0,

            "authorities": [
                {
                    "publicKey": "0x...",
                    "weight": 0,
                },
            ],

            "scheduledChange": {
                "delay": 0,
                "authorities": [
                    {
                        "publicKey": "0x...",
                        "weight": 0,
                    },
                ],
            },
        }
    }
}
```

- `finalized_block_header` is a hexadecimal-encoded string representing the hash of highest known finalized block.

- `consensus` is an _optional_ JSON object containing the consensus information of the chain.  
  If the chain does not have a consensus algorithm, this field is not present.  
  This object has the following format:

  - `consensusType` is a string indicating the consensus algorithm used by the chain.  
    This field can be one of the following values:

    - `"Aura"`: A string indicating the chain uses the Aura consensus algorithm.
    - `"Babe"`: A string indicating the chain uses the Babe consensus algorithm.
    - `"AllAuthorized"`: A string indicating the any node of the chain is allowed to produce blocks.
    The user is encourage to limit, through other means, the number of blocks that being accepted.

  - `aura` is an _optional_ JSON object containing the Aura consensus information of the chain.  
    This field is only present if the `consensusType` is `"Aura"`.  
    This object has the following format:

    - `authorities` is an array of JSON objects representing the list of authorities that must validate children of the block referred to by `finalized_block_header`.  
      Each object has the following format:
      - `publicKey` is a hexadecimal-encoded string representing the Sr25519 public key of the authority.
      - `slot_duration` is an unsigned integer indicating the slot duration of the chain in milliseconds.

  - `babe` is an _optional_ JSON object containing the Babe consensus information of the chain.  
    This field is only present if the `consensusType` is `"Babe"`.
    This object has the following format:

    - `slotPerEpoch` is an unsigned integer indicating the number of slots per epoch.  
      This field is configured at the genesis block and is constant for the lifetime of the chain.
    - `finalizedCurrentEpoch` is an _optional_ JSON object the information about the epoch the finalized block belongs to.  
      This field is not present if and only if the finalized block is the first block of the chain.  
      Epoch number zero starts at the block number one.  
      This object has the following format:
      - `epochIndex` is an unsigned integer indicating the index of the epoch.
        The epoch index is zero-based and is incremented by one for each new epoch.
      - `startSlotNumber` is an _optional_ unsigned integer indicating the slot at which the epoch starts.
        This field is not present if and only if the `epochIndex` is zero.
      - `authorities` is an array of JSON objects representing the authorities of the chain that are allowed to author blocks during this epoch.  
        Each object has the following format:
        - `publicKey` is a hexadecimal-encoded string representing the Sr25519 public key of the authority.
        - `weight` is an unsigned integer indicating the weight of the authority.
      - `randomness` is a hexadecimal-encoded string representing the randomness of the epoch.  
        This value is determined using the VRF output of the validators of the previous epoch.
      - `constant` is a JSON object containing the constant information of the epoch.  
        The constant represents a fraction, where the numerator is always inferior or equal to the denominator.  
        This object has the following format:
        - `numerator` is an unsigned integer indicating the numerator of the constant.
        - `denominator` is an unsigned integer indicating the denominator of the constant.
      - `allowedSlots` is a string indicating the allowed slots of the epoch.  
        This field can be one of the following values:
        - `"PrimarySlots"`: A string indicating the primary slots are allowed.
        - `"PrimaryAndSecondaryPlainSlots"`: A string indicating the primary and secondary plain slots are allowed.
        - `"PrimaryAndSecondaryVRFSlots"`: A string indicating the primary and secondary VRF slots are allowed.
    - `finalizedNextEpoch` is a JSON object containing the information about the next epoch.  
      This object has the same format as the `finalizedCurrentEpoch` object.

- `finality` is an _optional_ JSON object containing the finality information of the chain.  
  If the chain does not have a finality algorithm, this field is not present.  
  This object has the following format:

  - `finalityType` is a string indicating the finality algorithm used by the chain.  
    This field can be one of the following values:

    - `"Grandpa"`: A string indicating the chain uses the Grandpa finality algorithm.
    - `"Outsourced"`: A string indicating the chain uses the outsourced finality algorithm.

  - `grandpa` is an _optional_ JSON object containing the Grandpa finality information of the chain.  
    This field is only present if the `finalityType` is `"Grandpa"`.  
    This object has the following format:

    - `authoritiesSetId` is an unsigned integer indicating the id of the authorities set of the block right after the finalized block.
    - `authorities` is an array of JSON objects representing the authorities of the chain that are allowed to finalize blocks right after the finalized block.
      Each object has the following format:
      - `publicKey` is a hexadecimal-encoded string representing the Sr25519 public key of the authority.
      - `weight` is an unsigned integer indicating the weight of the authority.
    - `scheduledChange` is an _optional_ JSON object containing the information about the scheduled change of the authorities.  
      This field is not present if and only if the authorities set is not scheduled to change.  
      The block whose height is containing this field is still finalized using the authorities found in the above `authorities` field.
      This object has the following format:
      - `delay` is an unsigned integer indicating the delay in blocks before the authorities set is changed.
      - `authorities` is an array of JSON objects representing the new authorities set.  
        Each object has the following format:
        - `publicKey` is a hexadecimal-encoded string representing the Sr25519 public key of the authority.
        - `weight` is an unsigned integer indicating the weight of the authority.

#### Trusted Blocks

The `"trustedBlocks"` field is an array of JSON objects representing the expected hashes of blocks at given heights.  
This can be used to set trusted checkpoints. The client should refuse to import blocks with a different hash at the given height.  

```json
    "trustedBlocks": [
        {
            "blockNumber": "...",
            "blockHash": "0x...",
        }
    ]
```

- `"blockNumber"` is an unsigned integer represented as string indicating the block number.
- `"blockHash"` is a hexadecimal-encoded string representing the hash of the block at the given height.

#### Bad Blocks

The `"badBlocks"` field is an array of hexadecimal-encoded strings representing the hashes of blocks that should be considered invalid. These hashes represent known and unwanted blocks on forks that have been pruned.
