Data Structures
===

- [Data Structures](#data-structures)
- [Blockchain Data Structures](#blockchain-data-structures)
  - [Block](#block)
  - [Header](#header)
  - [AvailableDataHeader](#availabledataheader)
  - [AvailableData](#availabledata)
  - [EvidenceData](#evidencedata)
  - [Commit](#commit)
  - [Time](#time)
  - [BlockID](#blockid)
  - [HashDigest](#hashdigest)
  - [Address](#address)
  - [Evidence](#evidence)
  - [CommitSig](#commitsig)
  - [PublicKey](#publickey)
  - [Vote](#vote)
  - [Signature](#signature)
- [Serialization](#serialization)
- [Hashing](#hashing)
- [Public-Key Cryptography](#public-key-cryptography)
- [Merkle Trees](#merkle-trees)
  - [Binary Merkle Tree](#binary-merkle-tree)
  - [Sparse Binary Merkle Tree](#sparse-binary-merkle-tree)
  - [Namespace Merkle Tree](#namespace-merkle-tree)
    - [Verifying Merkle Proofs](#verifying-merkle-proofs)
- [Erasure Coding](#erasure-coding)
  - [TransactionData](#transactiondata)
  - [MessageData](#messagedata)
  - [Transaction](#transaction)
  - [Message](#message)
- [State](#state)

# Blockchain Data Structures

## Block

Blocks are the top-level data structure of the LazyLedger blockchain.

| name                  | type                                        | description                                                           |
| --------------------- | ------------------------------------------- | --------------------------------------------------------------------- |
| `header`              | [Header](#header)                           | Block header. Contains primarily identification info and commitments. |
| `availableDataHeader` | [AvailableDataHeader](#availabledataheader) | Header of available data. Contains commitments to erasure-coded data. |
| `availableData`       | [AvailableData](#availabledata)             | Data that is erasure-coded for availability.                          |
| `evidence`            | [EvidenceData](#evidencedata)               | Evidence used for slashing conditions (e.g. equivocation).            |
| `lastCommit`          | [Commit](#commit)                           | Previous block's Tendermint commit.                                   |

## Header

Block header, which is fully downloaded by both full clients and light clients.

| name                 | type                      | description                                                        |
| -------------------- | ------------------------- | ------------------------------------------------------------------ |
| `version`            | `uint64`                  | Version of the LazyLedger chain.                                   |
| `chainID`            | `uint64`                  | Chain ID. Each fork assigns itself a (unique) ID.                  |
| `height`             | `uint64`                  | Block height. The genesis block is at height `1`.                  |
| `time`               | [Time](#time)             | Timestamp of this block.                                           |
| `lastBlockID`        | [BlockID](#blockid)       | Previous block's ID.                                               |
| `lastCommitRoot`     | [HashDigest](#hashdigest) | Previous block's Tendermint commit root.                           |
| `validatorsRoot`     | [HashDigest](#hashdigest) | Validator set root for this block.                                 |
| `nextValidatorsRoot` | [HashDigest](#hashdigest) | Root of the next block's validator set.                            |
| `consensusHash`      | [HashDigest](#hashdigest) | Consensus parameters for this block.                               |
| `evidenceRoot`       | [HashDigest](#hashdigest) | Evidence data root.                                                |
| `availableDataRoot`  | [HashDigest](#hashdigest) | Root of [commitments to erasure-coded data](#availabledataheader). |
| `proposerAddress`    | [Address](#address)       | Address of this block's proposer.                                  |

## AvailableDataHeader

| name                       | type                          | description                            |
| -------------------------- | ----------------------------- | -------------------------------------- |
| `availableDataCommitments` | [HashDigest](#hashdigest)`[]` | Commitments to all erasure-coded data. |

## AvailableData

Data that is [erasure-coded](#erasure-coding) for [data availability checks](https://arxiv.org/abs/1809.09044).

| name                     | type                                | description                                     |
| ------------------------ | ----------------------------------- | ----------------------------------------------- |
| `transactionData`        | [TransactionData](#transactiondata) | Transaction data.                               |
| `intermediateStateRoots` | [HashDigest](#hashdigest)           | Intermediate state roots used for fraud proofs. |
| `messageData`            | [MessageData](#messagedata)         | Message data.                                   |

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

The block ID is a single Merkle root: the root of the [block header](#header)'s fields, in the order provided in the spec. The root is computed using a [binary Merkle tree](#binary-merkle-tree).

| name         | type                      | description                      |
| ------------ | ------------------------- | -------------------------------- |
| `headerRoot` | [HashDigest](#hashdigest) | Root of the block header fields. |

## HashDigest

| name      | type       | description           |
| --------- | ---------- | --------------------- |
| `rawData` | `byte[32]` | Raw hash digest data. |

Output of the [hashing](#hashing) function. Exactly 256 bits (32 bytes) long.

## Address

| name      | type       | description       |
| --------- | ---------- | ----------------- |
| `rawData` | `byte[20]` | Raw address data. |

Addresses are the last `20` bytes of the [hash](#hashing) [digest](#hashdigest) of the [public key](#publickey).

## Evidence

| name     | type                    | description |
| -------- | ----------------------- | ----------- |
| `pubKey` | [PublicKey](#publickey) |             |
| `voteA`  | [Vote](#vote)           |             |
| `voteB`  | [Vote](#vote)           |             |

## CommitSig

```C++
enum BlockIDFlag : uint8_t {
    BlockIDFlagAbsent = 1,
    BlockIDFlagCommit = 2,
    BlockIDFlagNil = 3,
};
```

| name               | type                    | description |
| ------------------ | ----------------------- | ----------- |
| `blockIDFlag`      | `BlockIDFlag`           |             |
| `validatorAddress` | [Address](#address)     |             |
| `timestamp`        | [Time](#time)           |             |
| `signature`        | [Signature](#signature) |             |

## PublicKey

| name | type       | description              |
| ---- | ---------- | ------------------------ |
| `x`  | `byte[32]` | `x` value of public key. |
| `y`  | `byte[32]` | `y` value of public key. |

## Vote

```C++
enum VoteType : uint8_t {
    Prevote = 1,
    Precommit = 2,
};
```

| name               | type                    | description |
| ------------------ | ----------------------- | ----------- |
| `type`             | `VoteType`              |             |
| `height`           | `uint64`                |             |
| `round`            | `uint64`                |             |
| `blockID`          | [BlockID](#blockid)     |             |
| `timestamp`        | [Time](#time)           |             |
| `validatorAddress` | [Address](#address)     |             |
| `validatorIndex`   | `uint64`                |             |
| `signature`        | [Signature](#signature) |             |

## Signature

| name | type       | description                                                          |
| ---- | ---------- | -------------------------------------------------------------------- |
| `r`  | `byte[32]` | `r` value of the signature.                                          |
| `vs` | `byte[32]` | 1-bit `v` value followed by last 255 bits of `s` value of signature. |

Output of the [signing](#public-key-cryptography) process.

# Serialization

Unless otherwise indicated explicitly, objects are serialized using [protobuf3](https://developers.google.com/protocol-buffers/docs/proto3).

# Hashing

All protocol-level hashing is done using [Keccak-256](https://keccak.team/keccak.html), and not SHA3-256 ([FIPS 202](https://keccak.team/specifications.html#FIPS_202)). This is to enable compatibility with [Ethereum](https://ethereum.org)'s EVM. Keccak-256 outputs a digest that is 256 bits (i.e. 32 bytes) long.

Libraries implementing Keccak-256 are available in Go (https://godoc.org/golang.org/x/crypto/sha3) and Rust (https://docs.rs/sha3).

Unless otherwise indicated explicitly, objects are first [serialized](#serialization) before being hashed.

# Public-Key Cryptography

Consensus-critical data is authenticated using [ECDSA](https://www.secg.org/sec1-v2.pdf), with the curve [secp256k1](https://en.bitcoin.it/wiki/Secp256k1). A highly-optimized library is available in C (https://github.com/bitcoin-core/secp256k1), with wrappers in Go (https://pkg.go.dev/github.com/ethereum/go-ethereum/crypto/secp256k1) and Rust (https://docs.rs/crate/secp256k1).

[Public keys](#publickey) are encoded in uncompressed form, as the concatenation of the `x` and `y` values. No prefix is needed to distinguish between encoding schemes as this is the only encoding supported.

Deterministic signatures ([RFC-6979](https://tools.ietf.org/rfc/rfc6979.txt)) should be used when signing, but this is not enforced at the protocol level as it cannot be.

[Signatures](#signature) are represented as the `r` and `s` (each 32 bytes), and `v` (1-bit) values of the signature. `r` and `s` take on their usual meaning (see: [SEC 1, 4.1.3 Signing Operation](https://www.secg.org/sec1-v2.pdf)), while `v` is used for recovering the public key from a signature more quickly (see: [SEC 1, 4.1.6 Public Key Recovery Operation](https://www.secg.org/sec1-v2.pdf)). Only low-`s` values in signatures are valid (i.e. `s <= secp256k1.n//2`); `s` can be replaced with `-s mod secp256k1.n` during the signing process if it is high. Given this, the first bit of `s` will always be `0`, and can be used to store the 1-bit `v` value.

`v` represents the parity of the `Y` component of the point, `0` for even and `1` for odd. The `X` component of the point is assumed to always be low, since [the possibility of it being high is negligible](https://bitcoin.stackexchange.com/a/38909).

Putting it all together, the encoding for signatures is:
```
|    32 bytes   ||           32 bytes           |
[256-bit r value][1-bit v value][255-bit s value]
```

This encoding scheme is derived from [EIP 2098: Compact Signature Representation](https://eips.ethereum.org/EIPS/eip-2098).

# Merkle Trees

Merkle trees are used to authenticate various pieces of data across the LazyLedger stack, including transactions, messages, the validator set, etc. This section provides an overview of the different tree types used, and specifies how to construct them.

## Binary Merkle Tree

Binary Merkle trees are constructed in the usual fashion, with leaves being hashed once to get leaf nodes and internal nodes being the hash of the concatenation of their children. Note that when hashing leaves , `0x00` is prepended, and when hashing internal nodes, `0x01` is prepended. This avoids a second-preimage attack [where internal nodes are presented as leaves](https://en.wikipedia.org/wiki/Merkle_tree#Second_preimage_attack).

For leaf node of message `m`:
```C++
v = h(0x00, serialize(m))
```

An exceptions is made, in the case of empty leaf nodes: the value of an empty leaf node is 32-byte zero, i.e. `0x0000000000000000000000000000000000000000000000000000000000000000`. This is used rather than duplicating the last node if there are an odd number of nodes in order to avoid [CVE-2012-2459](https://nvd.nist.gov/vuln/detail/CVE-2012-2459). Implicitly, trees are padded with empty nodes up to the next larger power of 2.

For internal node with children `l` and `r`:
```C++
v = h(0x01, l, r) = h(0x01, l.v, r.v)
```

## Sparse Binary Merkle Tree


## Namespace Merkle Tree

Messages in LazyLedger are associated with a provided _namespace ID_, which identifies the application (or applications) that will read these messages when parsing blocks. The Namespace Merkle Tree (NMT) is a variation of the [Merkle Interval Tree](https://eprint.iacr.org/2018/642), which is itself an extension of the [Merkle Sum Tree](https://bitcointalk.org/index.php?topic=845978.0).

Construction of a NMT is similar to that of a plain binary Merkle tree, but with a different hashing method that commits to intervals of namespace IDs.

For leaf node of message `m`:
```C++
n_min = m.namespace_id
n_max = m.namespace_id
v = h(serialize(m))
```

The `namespace_id` field is the namespace ID of the message, which is a [`NAMESPACE_ID_BYTES`](consensus.md#system-parameters)-byte unsigned integer.

Before being hashed, the [message](#message)s are [serialized](#serialization).

For internal node with children `l` and `r`:
```C++
n_min = min(l.n_min, r.n_min)
n_max = max(l.n_max, r.n_max)
v = h(l, r) = h(l.min, l.max, l.v, r.min, r.max, r.v)
```

### Verifying Merkle Proofs


# Erasure Coding



## TransactionData

Wrapper for transaction data, which is a simple list of [Transaction](#transaction)s.

 | name           | type                            | description           |
 | -------------- | ------------------------------- | --------------------- |
 | `transactions` | [Transaction](#transaction)`[]` | List of transactions. |

TODO define a Transaction format

## MessageData

Wrapper for message data, which is a simple list of [Message](#message)s.

 | name       | type                    | description       |
 | ---------- | ----------------------- | ----------------- |
 | `messages` | [Message](#message)`[]` | List of messages. |

TODO define a Message format

## Transaction

## Message

# State

TODO validator set repr
