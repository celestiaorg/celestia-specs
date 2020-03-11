Consensus Rules
===

- [Consensus Rules](#consensus-rules)
- [System Parameters](#system-parameters)
- [Leader Selection](#leader-selection)
- [Fork Choice](#fork-choice)
- [Block Validity](#block-validity)
  - [Formatting](#formatting)
  - [Availability](#availability)

# System Parameters

| name                   | type     | value | description                                       |
| ---------------------- | -------- | ----- | ------------------------------------------------- |
| `NAMESPACE_ID_BYTES`   | `uint64` | `32`  | Size of namespace ID, in bytes.                   |
| `SHARE_SIZE`           | `uint64` | `256` | Size of transaction and message shares, in bytes. |
| `SHARE_RESERVED_BYTES` | `uint64` | `1`   | Bytes reserved at the beginning of each share.    |

# Leader Selection

# Fork Choice

# Block Validity

## Formatting

Leaves in the message [Namespace Merkle Tree](data_structures.md#namespace-merkle-tree) must be ordered lexicographically by namespace ID.

## Availability
