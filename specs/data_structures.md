Data Structures
===

- [Data Structures](#data-structures)
- [Data Structures Overview](#data-structures-overview)
- [Blockchain Data Structures](#blockchain-data-structures)
  - [Block](#block)
  - [Header](#header)
  - [AvailableDataHeader](#availabledataheader)
  - [AvailableData](#availabledata)
  - [Commit](#commit)
  - [Time](#time)
  - [BlockID](#blockid)
  - [HashDigest](#hashdigest)
  - [Address](#address)
  - [EvidenceData](#evidencedata)
  - [CommitSig](#commitsig)
  - [Evidence](#evidence)
  - [PublicKey](#publickey)
  - [Vote](#vote)
  - [Signature](#signature)
- [Serialization](#serialization)
- [Hashing](#hashing)
- [Public-Key Cryptography](#public-key-cryptography)
- [Merkle Trees](#merkle-trees)
  - [Binary Merkle Tree](#binary-merkle-tree)
  - [Annotated Merkle Tree](#annotated-merkle-tree)
    - [Verifying Annotated Merkle Proofs](#verifying-annotated-merkle-proofs)
  - [Namespace Merkle Tree](#namespace-merkle-tree)
  - [Sparse Merkle Tree](#sparse-merkle-tree)
- [Erasure Coding](#erasure-coding)
  - [TransactionData](#transactiondata)
  - [WrappedTransaction](#wrappedtransaction)
  - [Transaction](#transaction)
  - [IntermediateStateRootData](#intermediatestaterootdata)
  - [WrappedIntermediateStateRoot](#wrappedintermediatestateroot)
  - [IntermediateStateRoot](#intermediatestateroot)
  - [MessageData](#messagedata)
  - [Message](#message)
- [State](#state)

# Data Structures Overview

![fig: Block data structures.](figures/block_data_structures.svg)

# Blockchain Data Structures

## Block

Blocks are the top-level data structure of the LazyLedger blockchain.

| name                  | type                                        | description                                                           |
| --------------------- | ------------------------------------------- | --------------------------------------------------------------------- |
| `header`              | [Header](#header)                           | Block header. Contains primarily identification info and commitments. |
| `availableDataHeader` | [AvailableDataHeader](#availabledataheader) | Header of available data. Contains commitments to erasure-coded data. |
| `availableData`       | [AvailableData](#availabledata)             | Data that is erasure-coded for availability.                          |
| `lastCommit`          | [Commit](#commit)                           | Previous block's Tendermint commit.                                   |

## Header

Block header, which is fully downloaded by both full clients and light clients.

| name                | type                      | description                                                                                  |
| ------------------- | ------------------------- | -------------------------------------------------------------------------------------------- |
| `version`           | `uint64`                  | Version of the LazyLedger chain.                                                             |
| `chainID`           | `uint64`                  | Chain ID. Each fork assigns itself a (unique) ID.                                            |
| `height`            | `uint64`                  | Block height. The genesis block is at height `1`.                                            |
| `time`              | [Time](#time)             | Timestamp of this block.                                                                     |
| `lastBlockID`       | [BlockID](#blockid)       | Previous block's ID.                                                                         |
| `lastCommitRoot`    | [HashDigest](#hashdigest) | Previous block's Tendermint commit root.                                                     |
| `consensusHash`     | [HashDigest](#hashdigest) | Consensus parameters for this block.                                                         |
| `stateCommitment`   | [HashDigest](#hashdigest) | Commitment to state root and validator set root after this block's transactions are applied. |
| `availableDataRoot` | [HashDigest](#hashdigest) | Root of [commitments to erasure-coded data](#availabledataheader).                           |
| `proposerAddress`   | [Address](#address)       | Address of this block's proposer.                                                            |

## AvailableDataHeader

| name                       | type                          | description                            |
| -------------------------- | ----------------------------- | -------------------------------------- |
| `availableDataCommitments` | [HashDigest](#hashdigest)`[]` | Commitments to all erasure-coded data. |

## AvailableData

Data that is [erasure-coded](#erasure-coding) for [data availability checks](https://arxiv.org/abs/1809.09044).

| name                        | type                                                    | description                                                                                                     |
| --------------------------- | ------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `transactionData`           | [TransactionData](#transactiondata)                     | Transaction data. Transactions modify the validator set and balances, and pay fees for messages to be included. |
| `intermediateStateRootData` | [IntermediateStateRootData](#intermediatestaterootdata) | Intermediate state roots used for fraud proofs.                                                                 |
| `evidenceData`              | [EvidenceData](#evidencedata)                           | Evidence used for slashing conditions (e.g. equivocation).                                                      |
| `messageData`               | [MessageData](#messagedata)                             | Message data. Messages are app data.                                                                            |

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

## EvidenceData

Wrapper for evidence data.

| name        | type                      | description                                    |
| ----------- | ------------------------- | ---------------------------------------------- |
| `evidences` | [Evidence](#evidence)`[]` | List of evidence used for slashing conditions. |

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

## Evidence

| name     | type                    | description |
| -------- | ----------------------- | ----------- |
| `pubKey` | [PublicKey](#publickey) |             |
| `voteA`  | [Vote](#vote)           |             |
| `voteB`  | [Vote](#vote)           |             |

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

Binary Merkle trees are constructed in the usual fashion, with leaves being hashed once to get leaf node values and internal node values being the hash of the concatenation of their children. The specific mechanism for hashing leaves for leaf nodes and children for internal nodes may be different (see: [annotated Merkle trees](#annotated-merkle-tree)), but for plain binary Merkle trees are the same.

For leaf node of leaf message `m`, its value `v` is:
```C++
v = h(serialize(m))
```

An exception is made, in the case of empty leaves: the value of a leaf node with an empty leaf is 32-byte zero, i.e. `0x0000000000000000000000000000000000000000000000000000000000000000`. This is used rather than duplicating the last node if there are an odd number of nodes (the [Bitcoin design](https://github.com/bitcoin/bitcoin/blob/5961b23898ee7c0af2626c46d5d70e80136578d3/src/consensus/merkle.cpp#L9-L43)) to avoid the complexities in that design, which resulted in e.g. [CVE-2012-2459](https://nvd.nist.gov/vuln/detail/CVE-2012-2459). By constructions, trees are implicitly padded with empty leaves up to the smallest enclosing power of 2.

For internal node with children `l` and `r`, its value `v` is:
```C++
v = h(l.v, r.v)
```

## Annotated Merkle Tree

Merkle trees can be augmented as generic annotated Merkle trees, where additional fields can be contained in each node. One of the early annotated Merkle trees is the [Merkle Sum Tree](https://bitcointalk.org/index.php?topic=845978.0), which allows for compact fraud proofs to be made of fees collected in a block.

Annotated Merkle trees have extra fields and methods to compute values for those fields, i.e. `f_1, ..., f_n, v` for `n` fields (note that if `n=0`, the annotated Merkle tree is a plain [binary Merkle tree](#binary-merkle-tree)). The value of field `f_i` is computed with the method `m_i_i(height, left_child_field, right_child_field)` for internal nodes and `m_i_l(message)` for leaf nodes.

For leaf node of leaf message `m`, its value `v` and fields `f_1, ..., f_n` are:
```C++
f_1 = m_1_l(m)
...
f_n = m_n_l(m)
v = h(serialize(m))
```

For internal node at height `height` with children `l` and `r`, its value `v` and fields `f_1, ..., f_n` are:
```C++
f_1 = m_1_i(height, l.f_1, r.f_1)
...
f_n = m_n_i(height, l.f_n, r.f_n)
v = h(l.f_1, ..., l.f_n, l.v, r.f_1, ..., r.f_n, r.v)
```

If a compact Merkle root is needed, the root level (which consists of root fields and a root value) can be hashed once.

As an example of annotation, when hashing leaves, `0x00` can be prepended, and when hashing internal nodes, `0x01` can be prepended (i.e. `m_1_l() = 0x00` and `m_1_i() = 0x01`). This avoids a second-preimage attack [where internal nodes are presented as leaves](https://en.wikipedia.org/wiki/Merkle_tree#Second_preimage_attack) for incomplete trees.

### Verifying Annotated Merkle Proofs

In addition to the root, leaf, index, and sibling values of a Merkle proof for a plain [binary Merkle tree](#binary-merkle-tree), Merkle proofs for annotated Mekle trees have the sibling field values. Proofs are verified by using the appropriate methods to compute field values.

## Namespace Merkle Tree

Messages in LazyLedger are associated with a provided _namespace ID_, which identifies the application (or applications) that will read these messages when parsing blocks. The Namespace Merkle Tree (NMT) is a variation of the [Merkle Interval Tree](https://eprint.iacr.org/2018/642).

The NMT is an annotated Merkle tree with two additional fields and methods that indicate the range of namespace IDs in each node's subtree.

For leaf node of message `m`:
```C++
n_min = m_1_l(m) = m.namespace_id
n_max = m_2_l(m) = m.namespace_id
v = h(serialize(m))
```

The `namespace_id` message field here is the namespace ID of the message, which is a [`NAMESPACE_ID_BYTES`](consensus.md#system-parameters)-byte unsigned integer.

Before being hashed, the [messages](#message) are [serialized](#serialization).

For internal node with children `l` and `r`:
```C++
n_min = m_1_i(height, l, r) = min(l.n_min, r.n_min)
n_max = m_2_i(height, l, r) = max(l.n_max, r.n_max)
v = h(l, r) = h(l.n_min, l.n_max, l.v, r.n_min, r.n_max, r.v)
```

## Sparse Merkle Tree

Sparse Merkle Trees (SMTs) are _sparse_, i.e. they contain mostly empty leaves. They can be used as key-value stores for arbitrary data, as each leaf is keyed by its index in the tree. Storage efficiency is achieved through clever use of implicit defaults, avoiding the need to store empty leaves.

Default values are given to leaf nodes with empty leaves. While this is sufficient to pre-compute the values of intermediate nodes that are roots of empty subtrees, a further simplification is to extend this default value to all nodes that are roots of empty subtrees. The 32-byte zero, i.e. `0x0000000000000000000000000000000000000000000000000000000000000000`, is used as the default value.

SMTs can further be extended with _compact_ proofs. [Merkle proofs](#verifying-annotated-merkle-proofs) are composed, among other things, of a list of sibling node values. We note that, since nodes that are roots of empty subtrees have known values (the default value), these values do not need to be provided explicitly; it is sufficient to simply identify which siblings in the Merkle branch are roots of empty subtrees, which can be done with one bit per sibling.

For a Merkle branch of height `h`, an `h`-bit value is appended to the proof. The lowest bit corresponds to the sibling of the leaf node, and each higher bit corresponds to the next parent. A value of `1` indicates that the next value in the list of values provided explicitly in the proof should be used, and a value of `0` indicates that the default value should be used.

A proof into an SMT is structured as:

| name               | type                          | description                                                                                     |
| ------------------ | ----------------------------- | ----------------------------------------------------------------------------------------------- |
| `root`             | [HashDigest](#hashdigest)     | Merkle root.                                                                                    |
| `leaf`             | `byte[]`                      | Leaf value.                                                                                     |
| `index`            | `byte[32]`                    | Index of the leaf.                                                                              |
| `siblings`         | [HashDigest](#hashdigest)`[]` | Sibling hash values.                                                                            |
| `includedSiblings` | `byte[32]`                    | Bitfield of explicitly included sibling hashes. The lowest bit corresponds the leaf node level. |

# Erasure Coding



## TransactionData

| name                  | type                                          | description                   |
| --------------------- | --------------------------------------------- | ----------------------------- |
| `wrappedTransactions` | [WrappedTransaction](#wrappedtransaction)`[]` | List of wrapped transactions. |

## WrappedTransaction

| name           | type          | description                                                                                     |
| -------------- | ------------- | ----------------------------------------------------------------------------------------------- |
| `index`        | `uint64`      | Index of this transaction in the list of wrapped transactions. This is needed for fraud proofs. |
| `transaction`  | `Transaction` | Actual transaction.                                                                             |
| `messageStart` | `uint64`      | _Optional_. Starting share of message this transaction pays for.                                |

## Transaction

| name                | type                          | description                                                                                                                                                                                                                                     |
| ------------------- | ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| TODO                |                               |                                                                                                                                                                                                                                                 |
| `messageSize`       | `uint64`                      | Size of message this transaction pays a fee for, in `byte`s. If this transaction does not pay a fee for a message, must be `0`.                                                                                                                 |
| `messageShareRoots` | [HashDigest](#hashdigest)`[]` | Merkle roots of an optional message that this transaction pays a fee to be included in the current block. Messages are split into shares and committed to here. Large messages can span across rows, which requires more roots to the provided. |


## IntermediateStateRootData

| name                            | type                                                              | description                               |
| ------------------------------- | ----------------------------------------------------------------- | ----------------------------------------- |
| `wrappedIntermediateStateRoots` | [WrappedIntermediateStateRoot](#wrappedintermediatestateroot)`[]` | List of wrapped intermediate state roots. |

## WrappedIntermediateStateRoot

| name                    | type         | description                                                                                                     |
| ----------------------- | ------------ | --------------------------------------------------------------------------------------------------------------- |
| `index`                 | `uint64`     | Index of this intermediate state root in the list of intermediate state roots. This is needed for fraud proofs. |
| `intermediateStateRoot` | `HashDigest` | Intermediate state root. Used for fraud proofs.                                                                 |

## IntermediateStateRoot



## MessageData

| name       | type                    | description       |
| ---------- | ----------------------- | ----------------- |
| `messages` | [Message](#message)`[]` | List of messages. |

## Message

| name      | type     | description        |
| --------- | -------- | ------------------ |
| `rawData` | `byte[]` | Raw message bytes. |

# State

TODO validator set repr
