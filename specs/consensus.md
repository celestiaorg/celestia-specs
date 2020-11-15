Consensus Rules
===

- [System Parameters](#system-parameters)
  - [Units](#units)
  - [Constants](#constants)
  - [Reserved Namespace IDs](#reserved-namespace-ids)
  - [Reserved State Subtree IDs](#reserved-state-subtree-ids)
  - [Rewards and Penalties](#rewards-and-penalties)
- [Leader Selection](#leader-selection)
- [Fork Choice](#fork-choice)
- [Block Validity](#block-validity)
- [Block Structure](#block-structure)
  - [`block.header`](#blockheader)
  - [`block.availableDataHeader`](#blockavailabledataheader)
  - [`block.lastCommit`](#blocklastcommit)
  - [`block.availableData`](#blockavailabledata)
- [State Transitions](#state-transitions)
  - [`block.availableData.evidenceData`](#blockavailabledataevidencedata)
  - [`block.availableData.transactionData`](#blockavailabledatatransactiondata)
  - [Validators and Delegations](#validators-and-delegations)

## System Parameters

### Units

| name | SI    | value   | description         |
| ---- | ----- | ------- | ------------------- |
| `1u` | `1u`  | `10**0` | `1` unit.           |
| `2u` | `k1u` | `10**3` | `1000` units.       |
| `3u` | `M1u` | `10**6` | `1000000` units.    |
| `4u` | `G1u` | `10**9` | `1000000000` units. |

### Constants

| name                                 | type     | value   | unit    | description                                                                                                                                                         |
| ------------------------------------ | -------- | ------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `AVAILABLE_DATA_ORIGINAL_SQUARE_MAX` | `uint64` |         | `share` | Maximum number of rows/columns of the original data [shares](data_structures.md#share) in [square layout](data_structures.md#arranging-available-data-into-shares). |
| `CHAIN_ID`                           | `uint64` | `1`     |         | Chain ID. Each chain assigns itself a (unique) ID.                                                                                                                  |
| `GENESIS_COIN_COUNT`                 | `uint64` | `10**8` | `4u`    | `(= 100000000)` Number of coins at genesis.                                                                                                                         |
| `GRAFFITI_BYTES`                     | `uint64` | `32`    | `byte`  | Maximum size of transaction graffiti, in bytes.                                                                                                                     |
| `MAX_VALIDATORS`                     | `uint16` | `64`    |         | Maximum number of active validators.                                                                                                                                |
| `NAMESPACE_ID_BYTES`                 | `uint64` | `8`     | `byte`  | Size of namespace ID, in bytes.                                                                                                                                     |
| `NAMESPACE_ID_MAX_RESERVED`          | `uint64` | `255`   |         | Value of maximum reserved namespace ID (inclusive). 1 byte worth of IDs.                                                                                            |
| `SHARE_RESERVED_BYTES`               | `uint64` | `1`     | `byte`  | Bytes reserved at the beginning of each [share](data_structures.md#share). Must be sufficient to represent `SHARE_SIZE`.                                            |
| `SHARE_SIZE`                         | `uint64` | `256`   | `byte`  | Size of transaction and message [shares](data_structures.md#share), in bytes.                                                                                       |
| `STATE_SUBTREE_RESERVED_BYTES`       | `uint64` | `1`     | `byte`  | Number of bytes reserved to identify state subtrees.                                                                                                                |
| `UNBONDING_DURATION`                 | `uint32` |         | `block` | Duration, in blocks, for unbonding a validator or delegation.                                                                                                       |
| `VERSION`                            | `uint64` | `1`     |         | Version of the LazyLedger chain. Breaking changes (hard forks) must update this parameter.                                                                          |

### Reserved Namespace IDs

| name                                   | type          | value                | description                                                                   |
| -------------------------------------- | ------------- | -------------------- | ----------------------------------------------------------------------------- |
| `TRANSACTION_NAMESPACE_ID`             | `NamespaceID` | `0x0000000000000001` | Transactions: requests that modify the state.                                 |
| `INTERMEDIATE_STATE_ROOT_NAMESPACE_ID` | `NamespaceID` | `0x0000000000000002` | Intermediate state roots, committed after every transaction.                  |
| `EVIDENCE_NAMESPACE_ID`                | `NamespaceID` | `0x0000000000000003` | Evidence: fraud proofs or other proof of slashable action.                    |
| `TAIL_PADDING_NAMESPACE_ID`            | `NamespaceID` | `0xFFFFFFFFFFFFFFFE` | Tail padding: padding after all messages to fill up the original data square. |
| `PARITY_SHARE_NAMESPACE_ID`            | `NamespaceID` | `0xFFFFFFFFFFFFFFFF` | Parity shares: extended shares in the available data matrix.                  |

### Reserved State Subtree IDs

| name                             | type             | value  |
| -------------------------------- | ---------------- | ------ |
| `ACCOUNTS_SUBTREE_ID`            | `StateSubtreeID` | `0x01` |
| `ACTIVE_VALIDATORS_SUBTREE_ID`   | `StateSubtreeID` | `0x02` |
| `INACTIVE_VALIDATORS_SUBTREE_ID` | `StateSubtreeID` | `0x03` |

### Rewards and Penalties

| name                 | type     | value | unit | description                      |
| -------------------- | -------- | ----- | ---- | -------------------------------- |
| `BASE_REWARD`        | `uint64` |       |      |                                  |
| `BASE_REWARD_FACTOR` | `uint64` |       |      | Factor that scales base rewards. |

## Leader Selection

TODO

## Fork Choice

TODO

## Block Validity

The validity of a newly-seen block, `block`, is determined by two components, detailed in subsequent sections:
1. [Block structure](#block-structure): whether the block header is valid, and data in a block is arranged into a valid and matching data root (i.e. syntax).
1. [State transition](#state-transitions): whether the application of transactions in the block produces a matching and valid state root (i.e. semantics).

## Block Structure

Before executing [state transitions](#state-transitions), the structure of the [block](./data_structures.md#block) must be verified.

The following block fields are acquired from the network and parsed (i.e. [deserialized](./data_structures.md#serialization)). If they cannot be parsed, the block is ignored but is not explicitly considered invalid by consensus rules. Further implications of ignoring a block are found in the [networking spec](./networking.md).
1. [block.header](./data_structures.md#header)
1. [block.availableDataHeader](./data_structures.md#availabledataheader)
1. [block.lastCommit](./data_structures.md#commit)

If the above fields are parsed successfully, the available data `block.availableData` is acquired in erasure-coded form as [a list of share rows](./networking.md#availabledata), then parsed. If it cannot be parsed, the block is ignored but not explicitly invalid, as above.

### `block.header`

The [block header](./data_structures.md#header) `block.header` (`header` for short) is the first thing that is downloaded from the new block, and commits to everything inside the block in some way. For previous block `prev` (if `prev` is not known, then the block is ignored), and previous block header `prev.header`, the following checks must be `true`:

`availableDataOriginalSquareSize` is computed as described [here](./data_structures.md#header).

1. `header.height` == `prev.header.height + 1`.
1. `header.timestamp` > `prev.header.timestamp`.
1. `header.lastBlockID` == the [block ID](./data_structures.md#blockid) of `prev`.
1. `header.lastCommitHash` == the [hash](./data_structures.md#hashing) of `lastCommit`.
1. `header.consensusRoot` == the value computed [here](./data_structures.md#consensus-parameters).
1. `header.stateCommitment` == the root of the state, computed [with the application of all state transitions in this block](#state-transitions).
1. `availableDataOriginalSquareSize` <= [`AVAILABLE_DATA_ORIGINAL_SQUARE_MAX`](#constants).
1. `header.availableDataRoot` == TODO available data root
1. `header.proposerAddress` == the [leader](#leader-selection) for `header.height`.

TODO define the genesis block

### `block.availableDataHeader`

The [available data header](./data_structures.md#availabledataheader)) `block.availableDataHeader` (`availableDataHeader` for short) is then processed. This commits to the available data, which is only downloaded after the [consensus commit](#blocklastcommit) is processed. The following checks must be `true`:

1. Length of `availableDataHeader.rowRoots` == `ceil(availableDataOriginalSharesUsed / availableDataOriginalSquareSize) + availableDataOriginalSquareSize`.
1. Length of `availableDataHeader.colRoots` == `availableDataOriginalSquareSize * 2`.
1. The length of each element in `availableDataHeader.rowRoots` and `availableDataHeader.colRoots` must be [`32`](./consensus.md#hashing).

### `block.lastCommit`

The last [commit](./data_structures.md#commit) `block.lastCommit` (`lastCommit` for short) is processed next. This is the Tendermint commit (i.e. polka of votes) _for the previous block_. For previous block `prev` and previous block header `prev.header`, the following checks must be `true`:

1. `lastCommit.height` == `prev.header.height`.
1. `lastCommit.round` >= `1`.
1. `lastCommit.blockID` == the [block ID](./data_structures.md#blockid) of `prev`.
1. Length of `lastCommit.signatures` <= [`MAX_VALIDATORS`](#constants).
1. Each of `lastCommit.signatures` must be a valid [CommitSig](./data_structures.md#commitsig)
1. The sum of the votes for `prev` in `lastCommit` must be at least 2/3 (rounded up) of the voting power of `prev`'s next validator set.

TODO define specifically how to validate commitsigs

### `block.availableData`

The block's [available data](./data_structures.md#availabledata) (analogous to transactions in contemporary blockchain designs) `block.availableData` (`availableData` for short) is finally processed. The [list of share rows](./networking.md#availabledata) is parsed into the [actual data structures](./data_structures.md#availabledata) using the reverse of [the process to encode available data into shares](./data_structures.md#arranging-available-data-into-shares); if parsing fails here, the block is invalid.

TODO define fraud proof for invalid block here

Once parsed, the following checks must be `true`:

1. The commitments of the [erasure-coded extended](./data_structures.md#2d-reed-solomon-encoding-scheme) `availableData` must match those in `header.availableDataHeader`. Implicitly, this means that both rows and columns must be ordered lexicographically by namespace ID since they are committed to in a [Namespace Merkle Tree](data_structures.md#namespace-merkle-tree).
1. Length of `availableData.intermediateStateRootData` == length of `availableData.transactionData` + length of `availableData.evidenceData`.

## State Transitions

Once the basic structure of the block [has been validated](#block-structure), state transitions must be applied to compute the new state and state root.

### `block.availableData.evidenceData`

Evidence is the first set of state transitions that are applied, and represent proof of validator misbehavior.

TODO process evidence

### `block.availableData.transactionData`

Once [evidence has been processed](#blockavailabledataevidencedata), transactions are applied to the state. Note that _transactions_ mutate the state (essentially, the validator set and minimal balances), while _messages_ do not. See [the architecture documentation](./architecture.md) for more info.

`block.availableData.transactionData` is simply a list of [WrappedTransaction](./data_structures.md#wrappedtransaction)s. For each wrapped transaction in this list, `wrappedTransaction`, with index `i` (starting from `0`), the following checks must be `true`:

1. `wrappedTransaction.index` == `i`.

For `wrappedTransaction`'s [transaction](./data_structures.md#transaction) `transaction`, the following checks must be `true`:

1. `transaction.signature` must be a [valid signature](./data_structures.md#public-key-cryptography) over `transaction.signedTransactionData`.

TODO add some logic for signing over implicit data, e.g. chain ID

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
