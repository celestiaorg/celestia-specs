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
  - [BlockID](#blockid)
  - [HashDigest](#hashdigest)
  - [Address](#address)
  - [Transaction](#transaction)
  - [Evidence](#evidence)
  - [CommitSig](#commitsig)
- [Serialization](#serialization)
- [Hashing](#hashing)
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

| name               | type                      | description |
| ------------------ | ------------------------- | ----------- |
| version            | [Version](#version)       |             |
| chainID            | `uint64`                  |             |
| height             | `uint64`                  |             |
| time               | [Time](#time)             |             |
| lastBlockID        | [BlockID](#blockid)       |             |
| lastCommitHash     | [HashDigest](#hashdigest) |             |
| dataHash           | [HashDigest](#hashdigest) |             |
| validatorsHash     | [HashDigest](#hashdigest) |             |
| nextValidatorsHash | [HashDigest](#hashdigest) |             |
| consensusHash      | [HashDigest](#hashdigest) |             |
| appHash            | [HashDigest](#hashdigest) |             |
| lastResultsHash    | [HashDigest](#hashdigest) |             |
| evidenceHash       | [HashDigest](#hashdigest) |             |
| proposerAddress    | [Address](#address)       |             |

## Block Data

 | name | type                            | description |
 | ---- | ------------------------------- | ----------- |
 | txs  | [Transaction](#transaction)`[]` |             |

TODO define a transaction format

## Evidence Data

| name     | type                      | description |
| -------- | ------------------------- | ----------- |
| evidence | [Evidence](#evidence)`[]` |             |

## Commit

| name       | type                        | description |
| ---------- | --------------------------- | ----------- |
| height     | `uint64`                    |             |
| round      | `uint64`                    |             |
| blockID    | [BlockID](#blockid)         |             |
| signatures | [CommitSig](#commitsig)`[]` |             |

## Version

| name              | type     | description |
| ----------------- | -------- | ----------- |
| blockchainVersion | `uint64` |             |
| appVersion        | `uint64` |             |

## Time

https://developers.google.com/protocol-buffers/docs/reference/csharp/class/google/protobuf/well-known-types/timestamp

## BlockID

## HashDigest

## Address

## Transaction

## Evidence

| name   | type                 | description |
| ------ | -------------------- | ----------- |
| pubKey | [PubicKey](#signing) |             |
| voteA  | [Vote](#vote)        |             |
| voteB  | [Vote](#vote)        |             |

## CommitSig

# Serialization

https://github.com/tendermint/go-amino

# Hashing

https://en.wikipedia.org/wiki/SHA-3

https://godoc.org/golang.org/x/crypto/sha3

# Merkle Tree



# Namespace Merkle Tree



# Erasure Coding

