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
  - [CodingRoots](#codingroots)
  - [Message](#message)
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
| `data`       | [Data](#data)                 | Message data.                                                         |
| `evidence`   | [EvidenceData](#evidencedata) | Evidence used for slashing conditions (e.g. equivocation).            |
| `lastCommit` | [Commit](#commit)             | Previous block's Tendermint commit.                                   |

## Header

Block header, which is fully downloaded by both full clients and light clients.

| name                 | type                        | description                                                 |
| -------------------- | --------------------------- | ----------------------------------------------------------- |
| `version`            | `uint64`                    | Version of the LazyLedger chain.                            |
| `chainID`            | `uint64`                    | Chain ID. Each fork assigns itself a (unique) ID.           |
| `height`             | `uint64`                    | Block height. The genesis block is at height `1`.           |
| `time`               | [Time](#time)               | Timestamp of this block.                                    |
| `lastBlockID`        | [BlockID](#blockid)         | Previous block's ID.                                        |
| `lastCommitRoot`     | [HashDigest](#hashdigest)   | Previous block's Tendermint commit root.                    |
| `dataRoot`           | [HashDigest](#hashdigest)   | Message data root.                                          |
| `validatorsRoot`     | [HashDigest](#hashdigest)   | Validator set root for this block.                          |
| `nextValidatorsHash` | [HashDigest](#hashdigest)   | Root of the next block's validator set.                     |
| `consensusHash`      | [HashDigest](#hashdigest)   | Consensus parameters for this block.                        |
| `appHash`            | [HashDigest](#hashdigest)   |                                                             |
| `lastResultsHash`    | [HashDigest](#hashdigest)   |                                                             |
| `evidenceRoot`       | [HashDigest](#hashdigest)   | Evidence data root.                                         |
| `codingRoots`        | [CodingRoots](#codingroots) | Commitments (i.e. Merkle roots) of the erasure-coded block. |
| `proposerAddress`    | [Address](#address)         | Address of this block's proposer.                           |

## Data

Wrapper for message (i.e. transaction) data, which is a simple list of [Message](#message)s.

 | name       | type                    | description       |
 | ---------- | ----------------------- | ----------------- |
 | `messages` | [Message](#message)`[]` | List of messages. |

TODO define a message format

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

LazyLedger uses the [Google Protobuf Timestamp](https://developers.google.com/protocol-buffers/docs/reference/csharp/class/google/protobuf/well-known-types/timestamp) format for timestamps, which represents time as seconds in UTC Epoch Time and nanoseconds.

## BlockID

The block ID is comprised of two distinct Merkle roots:
1. The root of the [block header](#header)'s fields, in the order provided in the spec.
1. The root of the complete [serialized](#serialization) block [split into parts](#todo). This is used at the network layer for securely gossiping parts of blocks.

| name          | type                            | description               |
| ------------- | ------------------------------- | ------------------------- |
| `headerRoot`  | [HashDigest](#hashdigest)       | Root of the block header. |
| `partsHeader` | [PartSetHeader](#partsetheader) | Parts header.             |

## HashDigest

## Address

## CodingRoots

## Message

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

| name     | type                      | description                      |
| -------- | ------------------------- | -------------------------------- |
| `length` | `uint32`                  | Number of parts.                 |
| `root`   | [HashDigest](#hashdigest) | Root of the block's split parts. |

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
Keccak-256 outputs a digest that is 256 bits (i.e. 32 bytes) long.

Libraries implementing Keccak-256 are available in Go (https://godoc.org/golang.org/x/crypto/sha3) and Rust (https://docs.rs/sha3).

# Signing



# Merkle Tree



## Namespace Merkle Tree



# Erasure Coding

