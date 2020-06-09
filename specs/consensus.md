Consensus Rules
===

- [Consensus Rules](#consensus-rules)
- [System Parameters](#system-parameters)
  - [Units](#units)
  - [Constants](#constants)
  - [Types](#types)
  - [Reserved Namespace IDs](#reserved-namespace-ids)
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

## Types

| name          | type     |
| ------------- | -------- |
| `NamespaceID` | `uint64` |

## Reserved Namespace IDs

| name                                   | type          | value                                                                |
| -------------------------------------- | ------------- | -------------------------------------------------------------------- |
| `TRANSACTION_NAMESPACE_ID`             | `NamespaceID` | `0x0000000000000000000000000000000000000000000000000000000000000001` |
| `INTERMEDIATE_STATE_ROOT_NAMESPACE_ID` | `NamespaceID` | `0x0000000000000000000000000000000000000000000000000000000000000002` |
| `EVIDENCE_NAMESPACE_ID`                | `NamespaceID` | `0x0000000000000000000000000000000000000000000000000000000000000003` |
| `PARITY_SHARE_NAMESPACE_ID`            | `NamespaceID` | `0xFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF` |

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

A transaction `tx` that requests a new validator initializes the [Validator](data_structures.md#validator) field of that account as follows:
| name              | value                    |
| ----------------- | ------------------------ |
| `status`          | `ValidatorStatus.Queued` |
| `stakedBalance`   | `tx.amount`              |
| `commissionRate`  | `tx.commissionRate`      |
| `delegatedCount`  | `0`                      |
| `votingPower`     | `tx.amount`              |
| `pendingRewards`  | `0`                      |
| `latestEntry`     | `PeriodEntry(0)`         |
| `unbondingHeight` | `0`                      |
| `isSlashed`       | `false`                  |

At the end of a block at the end of an epoch, the top `MAX_VALIDATORS` validators by voting power are or become active. For newly-bonded validators, their status is changed to bonded and height values initialized.
| name     | value                    |
| -------- | ------------------------ |
| `status` | `ValidatorStatus.Bonded` |

For validators that were bonded but are no longer (either by being outside the top validators or through a transaction that requests unbonding), they begin unbonding.
| name              | value                       |
| ----------------- | --------------------------- |
| `status`          | `ValidatorStatus.Unbonding` |
| `unbondingHeight` | `block.height + 1`          |

Once an unbonding validator has waited at least `UNBONDING_DURATION` blocks, they can be unbonded, collecting their reward:
| name            | value                                 |
| --------------- | ------------------------------------- |
| `status`        | `ValidatorStatus.Unbonded`            |
| `stakedBalance` | `0`                                   |
| `votingPower`   | `old.votingPower - old.stakedBalance` |

Every time a bonded validator's voting power changes (i.e. when a delegation is added or removed), or when a validator begins unbonding, the rewards per unit of voting power accumulated so far are calculated:
| name             | value                                                    |
| ---------------- | -------------------------------------------------------- |
| `pendingRewards` | `0`                                                      |
| `latestEntry`    | `old.latestEntry + old.pendingRewards / old.votingPower` |

A transaction `tx` that requests a new delegation first updates the target validator's voting power:
| name             | value                         |
| ---------------- | ----------------------------- |
| `delegatedCount` | `old.delegatedCount + 1`      |
| `votingPower`    | `old.votingPower + tx.amount` |

then initializes the [Delegation](data_structures.md#delegation) field of that account as follows:
| name              | value                     |
| ----------------- | ------------------------- |
| `status`          | `DelegationStatus.Bonded` |
| `validator`       | `tx.validator`            |
| `stakedBalance`   | `tx.amount`               |
| `beginEntry`      | `validator.latestEntry`   |
| `endEntry`        | `PeriodEntry(0)`          |
| `unbondingHeight` | `0`                       |

A transaction `tx` that requests withdrawing a delegation first updates the delegation field:
| name              | value                        |
| ----------------- | ---------------------------- |
| `status`          | `DelegationStatus.Unbonding` |
| `endEntry`        | `validator.latestEntry`      |
| `unbondingHeight` | `block.height + 1`           |

then updates the target validator's voting power:
| name             | value                                      |
| ---------------- | ------------------------------------------ |
| `delegatedCount` | `old.delegatedCount - 1`                   |
| `votingPower`    | `old.votingPower - delegation.votingPower` |

## Formatting

Leaves in the message [Namespace Merkle Tree](data_structures.md#namespace-merkle-tree) must be ordered lexicographically by namespace ID.

## Availability
