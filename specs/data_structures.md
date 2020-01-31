Data Structures
===

- [Data Structures](#data-structures)
- [Blockchain Data Structures](#blockchain-data-structures)
  - [Block](#block)
  - [Header](#header)
  - [Data](#data)
  - [EvidenceData](#evidencedata)
  - [Commit](#commit)
  - [Version](#version)
  - [Time](#time)
  - [BlockID](#blockid)
  - [HashDigest](#hashdigest)
  - [Address](#address)
  - [Transaction](#transaction)
  - [Evidence](#evidence)
  - [CommitSig](#commitsig)
  - [PartSetHeader](#partsetheader)
  - [Vote](#vote)
- [Serialization](#serialization)
- [Hashing](#hashing)
- [Signing](#signing)
- [Merkle Tree](#merkle-tree)
  - [Namespace Merkle Tree](#namespace-merkle-tree)
- [Erasure Coding](#erasure-coding)

# Blockchain Data Structures

## Block

| name         | type                          | description |
| ------------ | ----------------------------- | ----------- |
| `header`     | [Header](#header)             |             |
| `data`       | [Data](#data)                 |             |
| `evidence`   | [EvidenceData](#evidencedata) |             |
| `lastCommit` | [Commit](#commit)             |             |

## Header

| name                 | type                      | description |
| -------------------- | ------------------------- | ----------- |
| `version`            | [Version](#version)       |             |
| `chainID`            | `string`                  |             |
| `height`             | `uint64`                  |             |
| `time`               | [Time](#time)             |             |
| `lastBlockID`        | [BlockID](#blockid)       |             |
| `lastCommitHash`     | [HashDigest](#hashdigest) |             |
| `dataHash`           | [HashDigest](#hashdigest) |             |
| `validatorsHash`     | [HashDigest](#hashdigest) |             |
| `nextValidatorsHash` | [HashDigest](#hashdigest) |             |
| `consensusHash`      | [HashDigest](#hashdigest) |             |
| `appHash`            | [HashDigest](#hashdigest) |             |
| `lastResultsHash`    | [HashDigest](#hashdigest) |             |
| `evidenceHash`       | [HashDigest](#hashdigest) |             |
| `proposerAddress`    | [Address](#address)       |             |

## Data

 | name  | type                            | description |
 | ----- | ------------------------------- | ----------- |
 | `txs` | [Transaction](#transaction)`[]` |             |

TODO define a transaction format

## EvidenceData

| name       | type                      | description |
| ---------- | ------------------------- | ----------- |
| `evidence` | [Evidence](#evidence)`[]` |             |

## Commit

| name         | type                        | description |
| ------------ | --------------------------- | ----------- |
| `height`     | `uint64`                    |             |
| `round`      | `uint64`                    |             |
| `blockID`    | [BlockID](#blockid)         |             |
| `signatures` | [CommitSig](#commitsig)`[]` |             |

## Version

| name                | type     | description |
| ------------------- | -------- | ----------- |
| `blockchainVersion` | `uint64` |             |
| `appVersion`        | `uint64` |             |

## Time

https://developers.google.com/protocol-buffers/docs/reference/csharp/class/google/protobuf/well-known-types/timestamp

## BlockID

| name          | type                            | description |
| ------------- | ------------------------------- | ----------- |
| `hash`        | [HashDigest](#hashdigest)       |             |
| `partsHeader` | [PartSetHeader](#partsetheader) |             |

## HashDigest

## Address

## Transaction

## Evidence

| name     | type                 | description |
| -------- | -------------------- | ----------- |
| `pubKey` | [PubicKey](#signing) |             |
| `voteA`  | [Vote](#vote)        |             |
| `voteB`  | [Vote](#vote)        |             |

## CommitSig

```C++
enum BlockIDFlag {
    BlockIDFlagAbsent = 0x01,
    BlockIDFlagCommit = 0x02,
    BlockIDFlagNil = 0x03,
};
```

| name               | type                  | description |
| ------------------ | --------------------- | ----------- |
| `blockIDFlag`      | `BlockIDFlag`         |             |
| `validatorAddress` | [Address](#address)   |             |
| `timestamp`        | [Time](#time)         |             |
| `signature`        | [Signature](#signing) |             |


## PartSetHeader

| name     | type                      | description |
| -------- | ------------------------- | ----------- |
| `length` | `uint32`                  |             |
| `hash`   | [HashDigest](#hashdigest) |             |

## Vote

```C++
enum VoteType {
    Prevote = 1,
    Precommit = 2,
};
```

| name               | type                  | description |
| ------------------ | --------------------- | ----------- |
| `type`             | `VoteType`            |             |
| `height`           | `uint64`              |             |
| `round`            | `uint64`              |             |
| `blockID`          | [BlockID](#blockid)   |             |
| `timestamp`        | [Time](#time)         |             |
| `validatorAddress` | [Address](#address)   |             |
| `validatorIndex`   | `uint64`              |             |
| `signature`        | [Signature](#signing) |             |

# Serialization

https://developers.google.com/protocol-buffers/docs/proto3

# Hashing

https://en.wikipedia.org/wiki/SHA-3

https://godoc.org/golang.org/x/crypto/sha3

https://docs.rs/sha3

# Signing



# Merkle Tree



## Namespace Merkle Tree



# Erasure Coding

