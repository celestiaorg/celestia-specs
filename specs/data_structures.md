Data Structures
===

- [Data Structures](#data-structures)
- [Blockchain Data Structures](#blockchain-data-structures)
  - [Block](#block)
  - [Block Header](#block-header)
  - [Block Data](#block-data)
  - [Evidence Data](#evidence-data)
  - [Commit](#commit)
  - [Version](#version)
  - [Time](#time)
  - [Block ID](#block-id)
- [Address](#address)
- [Hash Digest](#hash-digest)
- [Merkle Tree](#merkle-tree)
- [Namespace Merkle Tree](#namespace-merkle-tree)
- [Erasure Coding](#erasure-coding)

# Blockchain Data Structures

## Block

| name       | type                           | description |
| ---------- | ------------------------------ | ----------- |
| header     | [Header](#block-header)        |             |
| data       | [Data](#block-data)            |             |
| evidence   | [EvidenceData](#evidence-data) |             |
| lastCommit | [Commit](#commit)              |             |

## Block Header

| name               | type                       | description |
| ------------------ | -------------------------- | ----------- |
| version            | [Version](#version)        |             |
| chainID            | `uint64`                   |             |
| height             | `uint64`                   |             |
| time               | [Time](#time)              |             |
| lastBlockID        | [BlockID](#block-id)       |             |
| lastCommitHash     | [HashDigest](#hash-digest) |             |
| dataHash           | [HashDigest](#hash-digest) |             |
| validatorsHash     | [HashDigest](#hash-digest) |             |
| nextValidatorsHash | [HashDigest](#hash-digest) |             |
| consensusHash      | [HashDigest](#hash-digest) |             |
| appHash            | [HashDigest](#hash-digest) |             |
| lastResultsHash    | [HashDigest](#hash-digest) |             |
| evidenceHash       | [HashDigest](#hash-digest) |             |
| proposerAddress    | [Address](#address)        |             |

## Block Data

 | name | type       | description |
 | ---- | ---------- | ----------- |
 | txs  | `byte[][]` |             |

TODO define a transaction format

## Evidence Data

## Commit

## Version

| name              | type     | description |
| ----------------- | -------- | ----------- |
| blockchainVersion | `uint64` |             |
| appVersion        | `uint64` |             |

## Time

## Block ID

# Address



# Hash Digest



# Merkle Tree



# Namespace Merkle Tree



# Erasure Coding

