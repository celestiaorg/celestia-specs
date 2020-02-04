Data Structures
===

- [Data Structures](#data-structures)
- [Blockchain Data Structures](#blockchain-data-structures)
  - [Block](#block)
  - [Header](#header)
  - [Data](#data)
  - [EvidenceData](#evidencedata)
  - [Commit](#commit)
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

Blocks are the top-level data structure of the LazyLedger blockchain.

| name         | type                          | description                                                           |
| ------------ | ----------------------------- | --------------------------------------------------------------------- |
| `header`     | [Header](#header)             | Block header. Contains primarily identification info and commitments. |
| `data`       | [Data](#data)                 | Transaction data.                                                     |
| `evidence`   | [EvidenceData](#evidencedata) | Evidence used for slashing conditions (e.g. equivocation).            |
| `lastCommit` | [Commit](#commit)             | Last block's Tendermint commit.                                       |

## Header

| name                 | type                      | description                                       |
| -------------------- | ------------------------- | ------------------------------------------------- |
| `version`            | `uint64`                  | Version of the LazyLedger chain.                  |
| `chainID`            | `uint64`                  | Chain ID. Each fork assigns itself a (unique) ID. |
| `height`             | `uint64`                  | Block height. The genesis block is at height `1`. |
| `time`               | [Time](#time)             |                                                   |
| `lastBlockID`        | [BlockID](#blockid)       |                                                   |
| `lastCommitHash`     | [HashDigest](#hashdigest) |                                                   |
| `dataHash`           | [HashDigest](#hashdigest) |                                                   |
| `validatorsHash`     | [HashDigest](#hashdigest) |                                                   |
| `nextValidatorsHash` | [HashDigest](#hashdigest) |                                                   |
| `consensusHash`      | [HashDigest](#hashdigest) |                                                   |
| `appHash`            | [HashDigest](#hashdigest) |                                                   |
| `lastResultsHash`    | [HashDigest](#hashdigest) |                                                   |
| `evidenceHash`       | [HashDigest](#hashdigest) |                                                   |
| `proposerAddress`    | [Address](#address)       |                                                   |

## Data

Wrapper for transaction data, which is a simple list of [Transaction](#transaction)s.

 | name  | type                            | description           |
 | ----- | ------------------------------- | --------------------- |
 | `txs` | [Transaction](#transaction)`[]` | List of transactions. |

TODO define a transaction format

## EvidenceData

Wrapper for evidence data.

| name       | type                      | description                                    |
| ---------- | ------------------------- | ---------------------------------------------- |
| `evidence` | [Evidence](#evidence)`[]` | List of evidence used for slashing conditions. |

## Commit

| name         | type                        | description |
| ------------ | --------------------------- | ----------- |
| `height`     | `uint64`                    |             |
| `round`      | `uint64`                    |             |
| `blockID`    | [BlockID](#blockid)         |             |
| `signatures` | [CommitSig](#commitsig)`[]` |             |

## Time

LazyLedger uses the [Google Protobuf Timestamp](https://developers.google.com/protocol-buffers/docs/reference/csharp/class/google/protobuf/well-known-types/timestamp) format for timestamps, which represents time as seconds in UTC Epoch Time and nonoseconds.


## BlockID

| name          | type                            | description |
| ------------- | ------------------------------- | ----------- |
| `hash`        | [HashDigest](#hashdigest)       |             |
| `partsHeader` | [PartSetHeader](#partsetheader) |             |

## HashDigest

## Address

## Transaction

## Evidence

| name     | type                  | description |
| -------- | --------------------- | ----------- |
| `pubKey` | [PublicKey](#signing) |             |
| `voteA`  | [Vote](#vote)         |             |
| `voteB`  | [Vote](#vote)         |             |

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

All protocol-level hashing is done using [Keccak-256](https://keccak.team/keccak.html), and not SHA3-256 ([FIPS 202](https://keccak.team/specifications.html#FIPS_202)).
This is to enable compatibility with [Ethereum](https://ethereum.org)'s EVM.

https://godoc.org/golang.org/x/crypto/sha3

https://docs.rs/sha3

# Signing



# Merkle Tree



## Namespace Merkle Tree



# Erasure Coding

