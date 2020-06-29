Consensus Rules
===

- [Consensus Rules](#consensus-rules)
- [System Parameters](#system-parameters)
  - [Units](#units)
  - [Constants](#constants)
  - [Type Aliases](#type-aliases)
  - [Reserved Namespace IDs](#reserved-namespace-ids)
  - [Reserved State Subtree IDs](#reserved-state-subtree-ids)
  - [Rewards and Penalties](#rewards-and-penalties)
- [Leader Selection](#leader-selection)
- [Fork Choice](#fork-choice)
- [Block Validity](#block-validity)
  - [State Transitions](#state-transitions)
    - [Validators and Delegations](#validators-and-delegations)
  - [Formatting](#formatting)
  - [Availability](#availability)

# System Parameters

## Units

| name | SI    | value   | description         |
| ---- | ----- | ------- | ------------------- |
| `1u` | `1u`  | `10**0` | `1` unit.           |
| `2u` | `k1u` | `10**3` | `1000` units.       |
| `3u` | `M1u` | `10**6` | `1000000` units.    |
| `4u` | `G1u` | `10**9` | `1000000000` units. |

## Constants

| name                                  | type     | value   | unit    | description                                                                                                                                                 |
| ------------------------------------- | -------- | ------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `VERSION`                             | `uint64` | `1`     |         | Version of the LazyLedger chain. Breaking changes (hard forks) must update this parameter.                                                                  |
| `CHAIN_ID`                            | `uint64` | `1`     |         | Chain ID. Each chain assigns itself a (unique) ID.                                                                                                          |
| `NAMESPACE_ID_BYTES`                  | `uint64` | `32`    | `byte`  | Size of namespace ID, in bytes.                                                                                                                             |
| `NAMESPACE_ID_MAX_RESERVED`           | `uint64` | `255`   |         | Value of maximum reserved namespace ID (inclusive). 1 byte worth of IDs.                                                                                    |
| `SHARE_SIZE`                          | `uint64` | `256`   | `byte`  | Size of transaction and message [shares](data_structures.md#share), in bytes.                                                                               |
| `SHARE_RESERVED_BYTES`                | `uint64` | `1`     | `byte`  | Bytes reserved at the beginning of each [share](data_structures.md#share). Must be sufficient to represent `SHARE_SIZE`.                                    |
| `AVAILABLE_DATA_ORIGINAL_SQUARE_SIZE` | `uint64` |         | `share` | Number of rows/columns of the original data [shares](data_structures.md#share) in [square layout](data_structures.md#arranging-available-data-into-shares). |
| `GENESIS_COIN_COUNT`                  | `uint64` | `10**8` | `4u`    | `(= 100000000)` Number of coins at genesis.                                                                                                                 |
| `UNBONDING_DURATION`                  | `uint32` |         | `block` | Duration, in blocks, for unbonding a validator or delegation.                                                                                               |
| `MAX_VALIDATORS`                      | `uint16` | `64`    |         | Maximum number of active validators.                                                                                                                        |
| `STATE_SUBTREE_RESERVED_BYTES`        | `uint64` | `1`     | `byte`  | Number of bytes reserved to identify state subtrees.                                                                                                        |

## Type Aliases

| name             | type                       |
| ---------------- | -------------------------- |
| `NamespaceID`    | `byte[NAMESPACE_ID_BYTES]` |
| `StateSubtreeID` | `byte`                     |

## Reserved Namespace IDs

| name                                   | type          | value                                                                |
| -------------------------------------- | ------------- | -------------------------------------------------------------------- |
| `TRANSACTION_NAMESPACE_ID`             | `NamespaceID` | `0x0000000000000000000000000000000000000000000000000000000000000001` |
| `INTERMEDIATE_STATE_ROOT_NAMESPACE_ID` | `NamespaceID` | `0x0000000000000000000000000000000000000000000000000000000000000002` |
| `EVIDENCE_NAMESPACE_ID`                | `NamespaceID` | `0x0000000000000000000000000000000000000000000000000000000000000003` |
| `TAIL_PADDING_NAMESPACE_ID`            | `NamespaceID` | `0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFE` |
| `PARITY_SHARE_NAMESPACE_ID`            | `NamespaceID` | `0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF` |

## Reserved State Subtree IDs

| name                             | type             | value  |
| -------------------------------- | ---------------- | ------ |
| `ACCOUNTS_SUBTREE_ID`            | `StateSubtreeID` | `0x01` |
| `ACTIVE_VALIDATORS_SUBTREE_ID`   | `StateSubtreeID` | `0x02` |
| `INACTIVE_VALIDATORS_SUBTREE_ID` | `StateSubtreeID` | `0x03` |


## Rewards and Penalties

| name                 | type     | value | unit | description                      |
| -------------------- | -------- | ----- | ---- | -------------------------------- |
| `BASE_REWARD`        | `uint64` |       |      |                                  |
| `BASE_REWARD_FACTOR` | `uint64` |       |      | Factor that scales base rewards. |

# Leader Selection

# Fork Choice

# Block Validity

## State Transitions

### Validators and Delegations

A transaction `tx` that requests a new validator initializes a new [Validator](data_structures.md#validator) leaf in the inactive validators subtree for that account as follows:
```
validator.status = ValidatorStatus.Queued
validator.stakedBalance = tx.amount
validator.commissionRate = tx.commissionRate
validator.delegatedCount = 0
validator.votingPower = tx.amount
validator.pendingRewards = 0
validator.latestEntry = PeriodEntry(0)
validator.unbondingHeight = 0
validator.isSlashed = false
```

At the end of a block at the end of an epoch, the top `MAX_VALIDATORS` validators by voting power are or become active (bonded). For newly-bonded validators, the entire validator object is moved to the active validators subtree and their status is changed to bonded.
```
validator.status = ValidatorStatus.Bonded
```

For validators that were bonded but are no longer (either by being outside the top `MAX_VALIDATORS` validators, through a transaction that requests unbonding, or by being slashing), the validator object is moved to the inactive validators subtree and they begin unbonding.
```
validator.status = ValidatorStatus.Unbonding
validator.unbondingHeight = block.height + 1
```

Once an unbonding validator has waited at least `UNBONDING_DURATION` blocks, they can be unbonded, collecting their reward:
```
old_stakedBalance = validator.stakedBalance

validator.status = ValidatorStatus.Unbonded
validator.stakedBalance = 0
validator.votingPower -= old_stakedBalance
```

Every time a bonded validator's voting power changes (i.e. when a delegation is added or removed), or when a validator begins unbonding, the rewards per unit of voting power accumulated so far are calculated:
```
old_pendingRewards = validator.pendingRewards
old_votingPower = validator.votingPower

validator.pendingRewards = 0
validator.latestEntry += old_pendingRewards / old_votingPower
```

A transaction `tx` that requests a new delegation first updates the target validator's voting power:
```
validator.delegatedCount += 1
validator.votingPower += tx.amount
```

then initializes the [Delegation](data_structures.md#delegation) field of that account as follows:
```
delegation.status = DelegationStatus.Bonded
delegation.validator = tx.to
delegation.stakedBalance = tx.amount
delegation.beginEntry = validator.latestEntry
delegation.endEntry = PeriodEntry(0)
delegation.unbondingHeight = 0
```

A transaction `tx` that requests withdrawing a delegation first updates the delegation field:
```
delegation.status = DelegationStatus.Unbonding
delegation.endEntry = validator.latestEntry
delegation.unbondingHeight = block.height + 1
```

then updates the target validator's voting power:
```
validator.delegatedCount -= 1
validator.votingPower -= delegation.votingPower
```

## Formatting

Leaves in the message [Namespace Merkle Tree](data_structures.md#namespace-merkle-tree) must be ordered lexicographically by namespace ID.

## Availability
