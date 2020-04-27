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

| name                          | type     | value   | unit   | description                                                                                  |
| ----------------------------- | -------- | ------- | ------ | -------------------------------------------------------------------------------------------- |
| `NAMESPACE_ID_BYTES`          | `uint64` | `64`    | `byte` | Size of namespace ID, in bytes.                                                              |
| `NAMESPACE_ID_RESERVED_BYTES` | `uint64` | `2`     | `byte` | Size of reserved namespace ID range, in bytes.                                               |
| `SHARE_SIZE`                  | `uint64` | `256`   | `byte` | Size of transaction and message shares, in bytes.                                            |
| `SHARE_RESERVED_BYTES`        | `uint64` | `1`     | `byte` | Bytes reserved at the beginning of each share. Must be sufficient to represent `SHARE_SIZE`. |
| `GENESIS_COIN_COUNT`          | `uint64` | `10**8` | `4u`   | `(= 100000000)` Number of coins at genesis.                                                  |

## Types

| name          | type     |
| ------------- | -------- |
| `NamespaceID` | `uint64` |

## Reserved Namespace IDs

| name                                   | type          | value                |
| -------------------------------------- | ------------- | -------------------- |
| `TRANSACTION_NAMESPACE_ID`             | `NamespaceID` | `0x0000000000000001` |
| `INTERMEDIATE_STATE_ROOT_NAMESPACE_ID` | `NamespaceID` | `0x0000000000000002` |
| `EVIDENCE_NAMESPACE_ID`                | `NamespaceID` | `0x0000000000000003` |
| `PARITY_SHARE_NAMESPACE_ID`            | `NamespaceID` | `0xFFFFFFFFFFFFFFFF` |

## Rewards and Penalties

| name                 | type     | value | unit | description                      |
| -------------------- | -------- | ----- | ---- | -------------------------------- |
| `BASE_REWARD`        | `uint64` |       |      |                                  |
| `BASE_REWARD_FACTOR` | `uint64` |       |      | Factor that scales base rewards. |

# Leader Selection

# Fork Choice

# Block Validity

## Formatting

Leaves in the message [Namespace Merkle Tree](data_structures.md#namespace-merkle-tree) must be ordered lexicographically by namespace ID.

## Availability
