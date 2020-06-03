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
    - [Calculating Rewards and Penalties](#calculating-rewards-and-penalties)
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
| name                            | value                    |
| ------------------------------- | ------------------------ |
| `status`                        | `ValidatorStatus.Queued` |
| `delegatedCount`                | `0`                      |
| `stakedBalance`                 | `tx.amount`              |
| `votingPower`                   | `tx.amount`              |
| `startHeight`                   | `0`                      |
| `heightOfLastVotingPowerChange` | `0`                      |
| `pendingRewards`                | `0`                      |
| `accumulatedVotingPower`        | `0`                      |
| `unbondingHeight`               | `0`                      |
| `commissionRate`                | `tx.commissionRate`      |
| `isSlashed`                     | `false`                  |

At the end of a block at the end of an epoch, the top `MAX_VALIDATORS` validators by voting power are or become active. For newly-bonded validators, their status is changed to bonded and height values initialized.
| name                            | value                    |
| ------------------------------- | ------------------------ |
| `status`                        | `ValidatorStatus.Bonded` |
| `startHeight`                   | `block.height + 1`       |
| `heightOfLastVotingPowerChange` | `block.height + 1`       |

For validators that were bonded but are no longer (either by being outside the top validators or through a transaction that requests unbonding), they begin unbonding.
| name              | value                       |
| ----------------- | --------------------------- |
| `status`          | `ValidatorStatus.Unbonding` |
| `unbondingHeight` | `block.height + 1`          |

Once an unbonding validator has waited at least `UNBONDING_DURATION` blocks, they can be unbonded, collecting their reward:
| name                     | value                                                                                                                                        |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `status`                 | `ValidatorStatus.Unbonded`                                                                                                                   |
| `stakedBalance`          | `0`                                                                                                                                          |
| `votingPower`            | `old.votingPower - old.stakedBalance`                                                                                                        |
| `pendingRewards`         | `old.pendingRewards - calculatedReward` (Calculated in [rewards and penalties](#calculating-rewards-and-penalties).)                         |
| `accumulatedVotingPower` | `old.accumulatedVotingPower - calculatedAccumulatedVotingPower` (Calculated in [rewards and penalties](#calculating-rewards-and-penalties).) |

Every time a bonded validator's voting power changes (i.e. when a delegation is added or removed), or when a validator begins unbonding, the rate at which accumulated voting power grows also changes. Intuitively, this "accumulated voting power" is similar to "coin days," but measures voting power over a number of blocks instead of coins over a number of days. The height of the last time the voting power of this validator was changed is updated to the current block height and the accumulated voting power is increased.
| name                            | value                                                                                               |
| ------------------------------- | --------------------------------------------------------------------------------------------------- |
| `heightOfLastVotingPowerChange` | `block.height`                                                                                      |
| `accumulatedVotingPower`        | `old.accumulatedVotingPower + old.votingPower * (block.height - old.heightOfLastVotingPowerChange)` |

A transactions `tx` that requests a new delegation first updates the target validator's voting power:
| name             | value                         |
| ---------------- | ----------------------------- |
| `delegatedCount` | `old.delegatedCount + 1`      |
| `votingPower`    | `old.votingPower + tx.amount` |
then initializes the [Delegation](data_structures.md#delegation) field of that account as follows:
| name              | value                     |
| ----------------- | ------------------------- |
| `status`          | `DelegationStatus.Bonded` |
| `validator`       | `tx.validator`            |
| `votingPower`     | `tx.amount`               |
| `startHeight`     | `block.height + 1`        |
| `unbondingHeight` | `0`                       |
| `pendingRewards`  | `0`                       |

A transaction `tx` that requests withdrawing a delegation first updates the delegation field:
| name              | value                                                                                           |
| ----------------- | ----------------------------------------------------------------------------------------------- |
| `status`          | `DelegationStatus.Unbonding`                                                                    |
| `unbondingHeight` | `block.height + 1`                                                                              |
| `pendingRewards`  | `calculatedReward` (Calculated in [rewards and penalties](#calculating-rewards-and-penalties).) |
then updates the target validator's voting power:
| name                     | value                                                                                                                                        |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `delegatedCount`         | `old.delegatedCount - 1`                                                                                                                     |
| `votingPower`            | `old.votingPower - delegation.votingPower`                                                                                                   |
| `pendingRewards`         | `old.pendingRewards - calculatedReward` (Same as above.)                                                                                     |
| `accumulatedVotingPower` | `old.accumulatedVotingPower - calculatedAccumulatedVotingPower` (Calculated in [rewards and penalties](#calculating-rewards-and-penalties).) |


### Calculating Rewards and Penalties

Due to the requirement that all incorrect state transitions be provable with a compact fraud proof that is cheap enough to verify within a smart contract on a remote chain, computing rewards and penalties must involve minimal or no iterations. The scheme presented here is inspired by Cosmos' [F1 fee distribution scheme](https://github.com/cosmos/cosmos-sdk/blob/master/docs/spec/_proposals/f1-fee-distribution/f1_fee_distr.pdf) and the concept of "[coin days](https://bitcointalk.org/index.php?topic=6172.msg90789#msg90789)."

F1 requires iterating over (potentially) a large number of blocks, but avoids needing to iterate over every delegation, all while being approximation-free. As such, it cannot be used for LazyLedger directly. The scheme presented here requires no iterations at all, but is not approximation-free. The intuition is that rewards of a delegation (or validator) are simply proportional to the accumulated voting power contributed by that delegation (or validator's stake) and the remaining accumulated voting power. This has the effect of "smoothing out" rewards, hence the lack of approximation-freeness.

Rewards with penalties for validators:

```
calculatedAccumulatedVotingPower = (block.height - validator.startHeight) * validator.stakedBalance
calculatedReward = validator.pendingRewards * calculatedAccumulatedVotingPower / validator.accumulatedVotingPower
if (validator.isSlashed)
    calculatedReward *= validator.slashRate
```

Rewards with penalties for delegations:
```
calculatedAccumulatedVotingPower = (block.height - delegation.startHeight) * delegation.votingPower
calculatedReward = validator.pendingRewards * calculatedAccumulatedVotingPower / validator.accumulatedVotingPower
if (validator.isSlashed)
    calculatedReward *= validator.slashRate
```

## Formatting

Leaves in the message [Namespace Merkle Tree](data_structures.md#namespace-merkle-tree) must be ordered lexicographically by namespace ID.

## Availability
