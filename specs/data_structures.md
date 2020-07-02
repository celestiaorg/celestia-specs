Data Structures
===

- [Data Structures](#data-structures)
- [Data Structures Overview](#data-structures-overview)
- [Type Aliases](#type-aliases)
- [Blockchain Data Structures](#blockchain-data-structures)
  - [Block](#block)
  - [Header](#header)
  - [AvailableDataHeader](#availabledataheader)
  - [AvailableData](#availabledata)
  - [Commit](#commit)
  - [Timestamp](#timestamp)
  - [BlockID](#blockid)
  - [HashDigest](#hashdigest)
  - [Address](#address)
  - [CommitSig](#commitsig)
  - [Signature](#signature)
- [Serialization](#serialization)
- [Hashing](#hashing)
- [Public-Key Cryptography](#public-key-cryptography)
- [Merkle Trees](#merkle-trees)
  - [Binary Merkle Tree](#binary-merkle-tree)
    - [Binary Merkle Tree Proofs](#binary-merkle-tree-proofs)
  - [Namespace Merkle Tree](#namespace-merkle-tree)
    - [Namespace Merkle Tree Proofs](#namespace-merkle-tree-proofs)
  - [Sparse Merkle Tree](#sparse-merkle-tree)
    - [Sparse Merkle Tree Proofs](#sparse-merkle-tree-proofs)
- [Erasure Coding](#erasure-coding)
  - [Reed-Solomon Erasure Coding](#reed-solomon-erasure-coding)
  - [2D Reed-Solomon Encoding Scheme](#2d-reed-solomon-encoding-scheme)
  - [Share](#share)
    - [Share Serialization](#share-serialization)
  - [Arranging Available Data Into Shares](#arranging-available-data-into-shares)
- [Available Data](#available-data)
  - [TransactionData](#transactiondata)
    - [WrappedTransaction](#wrappedtransaction)
    - [Transaction](#transaction)
    - [SignedTransactionData](#signedtransactiondata)
      - [SignedTransactionData: Transfer](#signedtransactiondata-transfer)
      - [SignedTransactionData: PayForMessage](#signedtransactiondata-payformessage)
      - [SignedTransactionData: PayForPadding](#signedtransactiondata-payforpadding)
      - [SignedTransactionData: CreateValidator](#signedtransactiondata-createvalidator)
      - [SignedTransactionData: BeginUnbondingValidator](#signedtransactiondata-beginunbondingvalidator)
      - [SignedTransactionData: UnbondValidator](#signedtransactiondata-unbondvalidator)
      - [SignedTransactionData: CreateDelegation](#signedtransactiondata-createdelegation)
      - [SignedTransactionData: BeginUnbondingDelegation](#signedtransactiondata-beginunbondingdelegation)
      - [SignedTransactionData: UnbondDelegation](#signedtransactiondata-unbonddelegation)
      - [SignedTransactionData: Burn](#signedtransactiondata-burn)
  - [IntermediateStateRootData](#intermediatestaterootdata)
    - [WrappedIntermediateStateRoot](#wrappedintermediatestateroot)
    - [IntermediateStateRoot](#intermediatestateroot)
  - [EvidenceData](#evidencedata)
    - [Evidence](#evidence)
    - [PublicKey](#publickey)
    - [Vote](#vote)
  - [MessageData](#messagedata)
    - [Message](#message)
- [State](#state)
  - [Account](#account)
  - [Delegation](#delegation)
  - [Validator](#validator)
  - [ActiveValidatorCount](#activevalidatorcount)
  - [PeriodEntry](#periodentry)
  - [Decimal](#decimal)
- [Consensus Parameters](#consensus-parameters)

# Data Structures Overview

![fig: Block data structures.](./figures/block_data_structures.svg)

# Type Aliases

| name                        | type                       |
| --------------------------- | -------------------------- |
| [`Address`](#address)       | `byte[20]`                 |
| `Amount`                    | `uint64`                   |
| [`BlockID`](#blockid)       | [HashDigest](#hashdigest)  |
| `FeeRate`                   | `uint64`                   |
| `Graffiti`                  | `NamespaceID`              |
| [`HashDigest`](#hashdigest) | `byte[32]`                 |
| `Height`                    | `uint64`                   |
| `NamespaceID`               | `byte[NAMESPACE_ID_BYTES]` |
| `Nonce`                     | `uint64`                   |
| `Round`                     | `uint64`                   |
| `StateSubtreeID`            | `byte`                     |
| [`Timestamp`](#timestamp)   | `uint64`                   |
| `VotingPower`               | `uint64`                   |

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

| name                | type                      | description                                                                  |
| ------------------- | ------------------------- | ---------------------------------------------------------------------------- |
| `height`            | [Height](#type-aliases)   | Block height. The genesis block is at height `1`.                            |
| `timestamp`         | [Timestamp](#timestamp)   | Timestamp of this block.                                                     |
| `lastBlockID`       | [BlockID](#blockid)       | Previous block's ID.                                                         |
| `lastCommitRoot`    | [HashDigest](#hashdigest) | Previous block's Tendermint commit root.                                     |
| `consensusRoot`     | [HashDigest](#hashdigest) | Merkle root of [consensus parameters](#consensus-parameters) for this block. |
| `stateCommitment`   | [HashDigest](#hashdigest) | The [state root](#state) after this block's transactions are applied.        |
| `availableDataRoot` | [HashDigest](#hashdigest) | Root of [commitments to erasure-coded data](#availabledataheader).           |
| `proposerAddress`   | [Address](#address)       | Address of this block's proposer.                                            |

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
| `height`     | [Height](#type-aliases)     |             |
| `round`      | [Round](#type-aliases)      |             |
| `blockID`    | [BlockID](#blockid)         |             |
| `signatures` | [CommitSig](#commitsig)`[]` |             |

## Timestamp

Timestamp is a [type alias](#type-aliases).

LazyLedger uses a 64-bit unsigned integer (`uint64`) to represent time in [TAI64](http://cr.yp.to/libtai/tai64.html) format.

## BlockID

BlockID is a [type alias](#type-aliases).

The block ID is a single Merkle root: the root of the [block header](#header)'s fields, in the order provided in the spec. The root is computed using a [binary Merkle tree](#binary-merkle-tree).

## HashDigest

HashDigest is a [type alias](#type-aliases).

Output of the [hashing](#hashing) function. Exactly 256 bits (32 bytes) long.

## Address

Address is a [type alias](#type-aliases).

Addresses are the last `20` bytes of the [hash](#hashing) [digest](#hashdigest) of the [public key](#publickey).

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
| `timestamp`        | [Timestamp](#timestamp) |             |
| `signature`        | [Signature](#signature) |             |

## Signature

| name | type       | description                                                          |
| ---- | ---------- | -------------------------------------------------------------------- |
| `r`  | `byte[32]` | `r` value of the signature.                                          |
| `vs` | `byte[32]` | 1-bit `v` value followed by last 255 bits of `s` value of signature. |

Output of the [signing](#public-key-cryptography) process.

# Serialization

Objects that are committed to or signed over require a canonical serialization. This is done using TODO.

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

Binary Merkle trees are constructed in the same fashion as described in [Certificate Transparency (RFC-6962)](https://tools.ietf.org/html/rfc6962). Leaves are hashed once to get leaf node values and internal node values are the hash of the concatenation of their children (either leaf nodes or other internal nodes).

The base case (an empty tree) is defined as zero:
```C++
v = 0x0000000000000000000000000000000000000000000000000000000000000000
```

For leaf node of leaf data `d`, its value `v` is:
```C++
v = h(0x00, serialize(d))
```

For internal node with children `l` and `r`, its value `v` is:
```C++
v = h(0x01, l.v, r.v)
```

Note that rather than duplicating the last node if there are an odd number of nodes (the [Bitcoin design](https://github.com/bitcoin/bitcoin/blob/5961b23898ee7c0af2626c46d5d70e80136578d3/src/consensus/merkle.cpp#L9-L43)), trees are allowed to be imbalanced. In other words, the height of each leaf may be different. For an example, see Section 2.1.3 of [Certificate Transparency (RFC-6962)](https://tools.ietf.org/html/rfc6962).

Leaves and internal nodes are hashed differently: the one-byte `0x00` is prepended for leaf nodes while `0x01` is prepended for internal nodes. This avoids a second-preimage attack [where internal nodes are presented as leaves](https://en.wikipedia.org/wiki/Merkle_tree#Second_preimage_attack) trees with leaves at different heights.

### Binary Merkle Tree Proofs

| name       | type                          | description                   |
| ---------- | ----------------------------- | ----------------------------- |
| `root`     | [HashDigest](#hashdigest)     | Merkle root.                  |
| `key`      | `byte[32]`                    | Key (i.e. index) of the leaf. |
| `siblings` | [HashDigest](#hashdigest)`[]` | Sibling hash values.          |
| `leaf`     | `byte[]`                      | Leaf value.                   |

## Namespace Merkle Tree

[Shares](#share) in LazyLedger are associated with a provided _namespace ID_. The Namespace Merkle Tree (NMT) is a variation of the [Merkle Interval Tree](https://eprint.iacr.org/2018/642), which is itself an extension of the [Merkle Sum Tree](https://bitcointalk.org/index.php?topic=845978.0). It allows for compact proofs around the inclusion or exclusion of shares with particular namespace IDs.

The base case (an empty tree) is defined as:
```C++
n_min = 0x0000000000000000000000000000000000000000000000000000000000000000
n_max = 0x0000000000000000000000000000000000000000000000000000000000000000
v = 0x0000000000000000000000000000000000000000000000000000000000000000
```

For leaf node of data `d`:
```C++
n_min = d.namespaceID
n_max = d.namespaceID
v = h(0x00, serialize(d)
```

The `namespaceID` message field here is the namespace ID of the leaf, which is a [`NAMESPACE_ID_BYTES`](consensus.md#system-parameters)-long byte array.

For internal node with children `l` and `r`:
```C++
n_min = min(l.n_min, r.n_min)
n_max = max(l.n_max, r.n_max)
v = h(l, r) = h(0x01, l.n_min, l.n_max, l.v, r.n_min, r.n_max, r.v)
```

### Namespace Merkle Tree Proofs

| name            | type                             | description                   |
| --------------- | -------------------------------- | ----------------------------- |
| `root`          | [HashDigest](#hashdigest)        | Merkle root.                  |
| `key`           | `byte[32]`                       | Key (i.e. index) of the leaf. |
| `siblingValues` | [HashDigest](#hashdigest)`[]`    | Sibling hash values.          |
| `siblingMins`   | [NamespaceID](#type-aliases)`[]` | Sibling min namespace IDs.    |
| `siblingMaxes`  | [NamespaceID](#type-aliases)`[]` | Sibling max namespace IDs.    |
| `leaf`          | `byte[]`                         | Leaf value.                   |

## Sparse Merkle Tree

Sparse Merkle Trees (SMTs) are _sparse_, i.e. they contain mostly empty leaves. They can be used as key-value stores for arbitrary data, as each leaf is keyed by its index in the tree. Storage efficiency is achieved through clever use of implicit defaults, avoiding the need to store empty leaves.

Additional rules are added on top of plain [binary Merkle trees](#binary-merkle-tree):
1. Default values are given to leaf nodes with empty leaves.
1. While the above rule is sufficient to pre-compute the values of intermediate nodes that are roots of empty subtrees, a further simplification is to extend this default value to all nodes that are roots of empty subtrees. The 32-byte zero, i.e. `0x0000000000000000000000000000000000000000000000000000000000000000`, is used as the default value. This rule takes precedence over the above one.
1. The number of hashing operations can be reduced to be logarithmic in the number of non-empty leaves on average, assuming a uniform distribution of non-empty leaf keys. An internal node that is the root of a subtree that contains exactly one non-empty leaf is replaced by that leaf's leaf node.

The base case (an empty tree) is defined as the default value:
```C++
v = 0x0000000000000000000000000000000000000000000000000000000000000000
```

For leaf node of leaf data `d` with key `k`, its value `v` is:
```C++
v = h(0x00, k, serialize(d))
```

The key of leaf nodes must be prepended, since the index of a leaf node that is not at the base of the tree cannot be determined without this information.

For internal node with children `l` and `r`, its value `v` is:
```C++
v = h(0x01, l.v, r.v)
```

### Sparse Merkle Tree Proofs

SMTs can further be extended with _compact_ proofs. [Merkle proofs](#verifying-annotated-merkle-proofs) are composed, among other things, of a list of sibling node values. We note that, since nodes that are roots of empty subtrees have known values (the default value), these values do not need to be provided explicitly; it is sufficient to simply identify which siblings in the Merkle branch are roots of empty subtrees, which can be done with one bit per sibling.

For a Merkle branch of height `h`, an `h`-bit value is appended to the proof. The lowest bit corresponds to the sibling of the leaf node, and each higher bit corresponds to the next parent. A value of `1` indicates that the next value in the list of values provided explicitly in the proof should be used, and a value of `0` indicates that the default value should be used.

A proof into an SMT is structured as:

| name               | type                          | description                                                                                     |
| ------------------ | ----------------------------- | ----------------------------------------------------------------------------------------------- |
| `root`             | [HashDigest](#hashdigest)     | Merkle root.                                                                                    |
| `key`              | `byte[32]`                    | Key (i.e. index) of the leaf.                                                                   |
| `depth`            | `uint16`                      | Depth of the leaf node. The root node is at depth `0`. Must be `<= 256`.                        |
| `siblings`         | [HashDigest](#hashdigest)`[]` | Sibling hash values.                                                                            |
| `includedSiblings` | `byte[32]`                    | Bitfield of explicitly included sibling hashes. The lowest bit corresponds the leaf node level. |
| `leaf`             | `byte[]`                      | Leaf value.                                                                                     |

# Erasure Coding

In order to enable trust-minimized light clients (i.e. light clients that do not rely on an honest majority of validating state assumption), it is critical that light clients can determine whether the data in each block is _available_ or not, without downloading the whole block itself. The technique used here was formally described in the paper [Fraud and Data Availability Proofs: Maximising Light Client Security and Scaling Blockchains with Dishonest Majorities](https://arxiv.org/abs/1809.09044).

The remainder of the subsections below specify the [2D Reed-Solomon erasure coding scheme](#2d-reed-solomon-encoding-scheme) used, along with the format of [shares](#share) and how [available data](#available-data) is arranged into shares.

## Reed-Solomon Erasure Coding

Note that while data is laid out in a two-dimensional square, rows and columns are erasure coded using a standard one-dimensional encoding.

Reed-Solomon erasure coding is used as the underlying coding scheme. The parameters are:
- 16-bit Galois field
- `AVAILABLE_DATA_ORIGINAL_SQUARE_SIZE` original pieces
- `AVAILABLE_DATA_ORIGINAL_SQUARE_SIZE` parity pieces (i.e `AVAILABLE_DATA_ORIGINAL_SQUARE_SIZE * 2` total pieces), for an erasure efficiency of 50%. In other words, any 50% of the pieces from the `AVAILABLE_DATA_ORIGINAL_SQUARE_SIZE * 2` total pieces are enough to recover the original data.
- `SHARE_SIZE` bytes per piece

[Leopard-RS](https://github.com/catid/leopard) is a C library that implements the above scheme with quasilinear runtime.

## 2D Reed-Solomon Encoding Scheme

The 2-dimensional data layout is described in this section. The roots of [NMTs](#namespace-merkle-tree) for each row and column across four quadrants of data in a `2k * 2k` matrix of shares, `Q0` to `Q3` (shown below), must be computed. In other words, `2k` row roots and `2k` column roots must be computed. The row and column roots are stored in the `availableDataCommitments` of the [AvailableDataHeader](#availabledataheader).

![fig: RS2D encoding: data quadrants.](./figures/rs2d_quadrants.svg)

The data of `Q0` is the original data, and the remaining quadrants are parity data. Setting `k = AVAILABLE_DATA_ORIGINAL_SQUARE_SIZE`, the original data first must be [split into shares](#share) and [arranged into a `k * k` matrix](#arranging-available-data-into-shares). Then the parity data can be computed.

Where `A -> B` indicates that `B` is computed using [erasure coding](#reed-solomon-erasure-coding) from `A`:
- `Q0 -> Q1` for each row in `Q0` and `Q1`
- `Q0 -> Q2` for each column in `Q0` and `Q2`
- `Q2 -> Q3` for each row in `Q2` and `Q3`

![fig: RS2D encoding: extending data.](./figures/rs2d_extending.svg)

As an example, the parity data in the second column of `Q2` (in striped purple) is computed by [extending](#reed-solomon-erasure-coding) the original data in the second column of `Q0` (in solid blue).

![fig: RS2D encoding: extending a column.](./figures/rs2d_extend.svg)

Now that all four quadrants of the `2k * 2k` matrix are filled, the row and column roots can be computed. To do so, each row/column is used as the leaves of a [NMT](#namespace-merkle-tree), for which the compact root is computed (i.e. an extra hash operation is used to produce a single [HashDigest](#hashdigest)). In this example, the fourth row root value is computed as the NMT root of the fourth row of `Q0` and the fourth row of `Q1` as leaves.

![fig: RS2D encoding: a row root.](./figures/rs2d_row.svg)

Finally, the `availableDataRoot` of the block [Header](#header) is computed as the Merkle root of the [binary Merkle tree](#binary-merkle-tree) with the row and column roots as leaves.

![fig: Available data root.](./figures/data_root.svg)

## Share

A share is a fixed-size data chunk that will be erasure-coded and committed to in [Namespace Merkle trees](#namespace-merkle-tree).

| name          | type                         | description                |
| ------------- | ---------------------------- | -------------------------- |
| `namespaceID` | [NamespaceID](#type-aliases) | Namespace ID of the share. |
| `rawData`     | `byte[SHARE_SIZE]`           | Raw share data.            |

An example layout of the share's internal bytes is shown below. For non-parity shares _with a reserved namespace_, the first `SHARE_RESERVED_BYTES` bytes (`*` in the figure) is the starting byte of the first request in the share as an unsigned integer, or `0` if there is none. In this example, the first byte would be `80` (or `0x50` in hex). For shares _with a non-reserved namespace_ (and parity shares), the first `SHARE_RESERVED_BYTES` bytes have no special meaning and are simply used to store data like all the other bytes in the share.

![fig: Reserved share.](./figures/share.svg)

For non-parity shares, if there is insufficient request data to fill the share, the remaining bytes are padded with `0`.

### Share Serialization

Shares [canonically serialized](#serialization) using only the raw share data, i.e. `serialize(share) = serialize(share.rawData)`.

## Arranging Available Data Into Shares

The previous sections described how some original data, arranged into a `k * k` matrix, can be extended into a `2k * 2k` matrix and committed to with NMT roots. This section specifies how [available data](#available-data) (which includes [transactions](#transactiondata), [intermediate state roots](#intermediatestaterootdata), [evidence](#evidencedata), and [messages](#messagedata)) is arranged into the matrix in the first place.

 First, for each of `transactionData`, `intermediateStateRootData`, and `evidenceData`, [serialize](#serialization) the data and split it up into `SHARE_SIZE-SHARE_RESERVED_BYTES`-byte [shares](#share). This data has a _reserved_ namespace ID, and as such the first `SHARE_RESERVED_BYTES` bytes for these shares has special meaning. Then, concatenate the lists of shares in the order: transactions, intermediate state roots, evidence. Note that by construction, each share only has a single namespace, and that the list of concatenated shares is [lexicographically ordered by namespace ID](consensus.md#reserved-namespace-ids).

These shares are arranged in the [first quadrant](#2d-reed-solomon-encoding-scheme) (`Q0`) of the `AVAILABLE_DATA_ORIGINAL_SQUARE_SIZE*2 * AVAILABLE_DATA_ORIGINAL_SQUARE_SIZE*2` available data matrix in _row-major_ order. In the example below, each reserved data element takes up exactly one share.

![fig: Original data: reserved.](./figures/rs2d_originaldata_reserved.svg)

Each message in the list `messageData` is _independently_ serialized and split into `SHARE_SIZE`-byte shares. For each message, it is placed in the available data matrix, with row-major order, as follows:
1. Place the first share of the message at the next unused location in the matrix, then place the remaining shares in the following locations.

Transactions [must commit to a Merkle root of a list of hashes](#transaction) that are each guaranteed (assuming the block is valid) to be subtree roots in one or more of the row NMTs. For additional info, see [the rationale document](../rationale/message_block_layout.md) for this section.

However, with only the rule above, interaction between the block producer and transaction sender is required to compute a commitment to the message the transaction sender can sign over. To remove interaction, messages can also be laid out using a non-interactive default:
1. Place the first share of the message at the next unused location in the matrix whose column in aligned with the largest power of 2 that is not larger than the message length or `AVAILABLE_DATA_ORIGINAL_SQUARE_SIZE`, then place the remaining shares in the following locations **unless** there are insufficient unused locations in the row.
1. If there are insufficient unused locations in the row, place the first share of the message at the first column of the next row. Then place the remaining shares in the following locations. By construction, any message whose length is greater than `AVAILABLE_DATA_ORIGINAL_SQUARE_SIZE` will be placed in this way.

In the example below, two messages (of lengths 2 and 1, respectively) are placed using the aforementioned default non-interactive rules.

![fig: Original data: messages.](./figures/rs2d_originaldata_message.svg)

The non-interactive default rules may introduce empty shares that do not belong to any message (in the example above, the top-right share is empty). These must be explicitly disclaimed by the block producer using [special transactions](#signedtransactiondata-payforpadding).

# Available Data

## TransactionData

| name                  | type                                          | description                   |
| --------------------- | --------------------------------------------- | ----------------------------- |
| `wrappedTransactions` | [WrappedTransaction](#wrappedtransaction)`[]` | List of wrapped transactions. |

### WrappedTransaction

Wrapped transactions include additional metadata by the block proposer that is committed to in the [available data matrix](#arranging-available-data-into-shares).

| name                | type                        | description                                                                                                                                                                                                                                                                                                |
| ------------------- | --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `index`             | `uint64`                    | Index of this transaction in the list of wrapped transactions. This information is lost when splitting transactions into [fixed-sized shares](#share), and needs to be re-added here for fraud proof support. Allows linking a transaction to an [intermediate state root](#wrappedintermediatestateroot). |
| `transaction`       | [Transaction](#transaction) | Actual transaction.                                                                                                                                                                                                                                                                                        |
| `messageStartIndex` | `uint64`                    | _Optional, only used if transaction pays for a message or padding_. Share index (in row-major order) of first share of message this transaction pays for. Needed for light verification of proper message inclusion.                                                                                       |

### Transaction

| name                    | type                                            | description                       |
| ----------------------- | ----------------------------------------------- | --------------------------------- |
| `signedTransactionData` | [SignedTransactionData](#signedtransactiondata) | Data payload that is signed over. |
| `signature`             | [Signature](#signature)                         | Signature.                        |

### SignedTransactionData

```C++
enum TransactionType : uint8_t {
    Transfer = 1,
    PayForMessage = 2,
    PayForPadding = 3,
    CreateValidator = 4,
    BeginUnbondingValidator = 5,
    UnbondValidator = 6,
    CreateDelegation = 7,
    BeginUnbondingDelegation = 8,
    UnbondDelegation = 9,
    Burn = 10,
};
```

Signed transaction data comes in a number of types:
1. [Transfer](#signedtransactiondata-transfer)
1. [PayForMessage](#signedtransactiondata-payformessage)
1. [PayForPadding](#signedtransactiondata-payforpadding)
1. [CreateValidator](#signedtransactiondata-createvalidator)
1. [BeginUnbondingValidator](#signedtransactiondata-beginunbondingvalidator)
1. [UnbondValidator](#signedtransactiondata-unbondvalidator)
1. [CreateDelegation](#signedtransactiondata-createdelegation)
1. [BeginUnbondingDelegation](#signedtransactiondata-beginunbondingdelegation)
1. [UnbondDelegation](#signedtransactiondata-unbonddelegation)
1. [Burn](#signedtransactiondata-burn)

Common fields are denoted here to avoid repeating descriptions:

| name         | type                     | description                                                                |
| ------------ | ------------------------ | -------------------------------------------------------------------------- |
| `type`       | `TransactionType`        | Type of the transaction. Each type indicates a different state transition. |
| `amount`     | [Amount](#type-aliases)  | Amount of coins to send, in `1u`.                                          |
| `to`         | [Address](#address)      | Recipient's address.                                                       |
| `maxFeeRate` | [FeeRate](#type-aliases) | The maximum fee rate the sender is willing to pay.                         |
| `nonce`      | [Nonce](#type-aliases)   | Nonce of sender.                                                           |

#### SignedTransactionData: Transfer

| name         | type                     | description                         |
| ------------ | ------------------------ | ----------------------------------- |
| `type`       | `TransactionType`        | Must be `TransactionType.Transfer`. |
| `amount`     | [Amount](#type-aliases)  |                                     |
| `to`         | [Address](#address)      |                                     |
| `maxFeeRate` | [FeeRate](#type-aliases) |                                     |
| `nonce`      | [Nonce](#type-aliases)   |                                     |

Transfers `amount` coins to `to`.

#### SignedTransactionData: PayForMessage

| name                     | type                           | description                                                  |
| ------------------------ | ------------------------------ | ------------------------------------------------------------ |
| `type`                   | `TransactionType`              | Must be `TransactionType.PayForMessage`.                     |
| `maxFeeRate`             | [FeeRate](#type-aliases)       |                                                              |
| `nonce`                  | [Nonce](#type-aliases)         |                                                              |
| `messageNamespaceID`     | [`NamespaceID`](#type-aliases) | Namespace ID of message this transaction pays a fee for.     |
| `messageSize`            | `uint64`                       | Size of message this transaction pays a fee for, in `byte`s. |
| `messageShareCommitment` | [HashDigest](#hashdigest)      | Commitment to message shares (details below).                |

Pays for the inclusion of a [message](#message) in the same block.

The commitment to message shares `messageShareCommitment` is a [Merkle root](#binary-merkle-tree) of message share roots. Each message share root is [a subtree root in a row NMT](#arranging-available-data-into-shares). For rationale, see [rationale doc](../rationale/message_block_layout.md).

#### SignedTransactionData: PayForPadding

| name                 | type                           | description                                                  |
| -------------------- | ------------------------------ | ------------------------------------------------------------ |
| `type`               | `TransactionType`              | Must be `TransactionType.PayForPadding`.                     |
| `messageNamespaceID` | [`NamespaceID`](#type-aliases) | Namespace ID of padding this transaction pays a fee for.     |
| `messageSize`        | `uint64`                       | Size of padding this transaction pays a fee for, in `byte`s. |

Pays for the inclusion of a padding shares in the same block. Padding shares are used between real messages that are not tightly packed. For rationale, see [rationale doc](../rationale/message_block_layout.md).

#### SignedTransactionData: CreateValidator

| name             | type                     | description                                |
| ---------------- | ------------------------ | ------------------------------------------ |
| `type`           | `TransactionType`        | Must be `TransactionType.CreateValidator`. |
| `amount`         | [Amount](#type-aliases)  |                                            |
| `maxFeeRate`     | [FeeRate](#type-aliases) |                                            |
| `nonce`          | [Nonce](#type-aliases)   |                                            |
| `commissionRate` | [Decimal](#decimal)      |                                            |

Create a new [Validator](#validator) at this address for `amount` coins worth of voting power.

#### SignedTransactionData: BeginUnbondingValidator

| name         | type                     | description                                        |
| ------------ | ------------------------ | -------------------------------------------------- |
| `type`       | `TransactionType`        | Must be `TransactionType.BeginUnbondingValidator`. |
| `maxFeeRate` | [FeeRate](#type-aliases) |                                                    |
| `nonce`      | [Nonce](#type-aliases)   |                                                    |

Begin unbonding the [Validator](#validator) at this address.

#### SignedTransactionData: UnbondValidator

| name         | type                     | description                                |
| ------------ | ------------------------ | ------------------------------------------ |
| `type`       | `TransactionType`        | Must be `TransactionType.UnbondValidator`. |
| `maxFeeRate` | [FeeRate](#type-aliases) |                                            |
| `nonce`      | [Nonce](#type-aliases)   |                                            |

Finish unbonding the [Validator](#validator) at this address.

#### SignedTransactionData: CreateDelegation

| name         | type                     | description                                 |
| ------------ | ------------------------ | ------------------------------------------- |
| `type`       | `TransactionType`        | Must be `TransactionType.CreateDelegation`. |
| `amount`     | [Amount](#type-aliases)  |                                             |
| `to`         | [Address](#address)      |                                             |
| `maxFeeRate` | [FeeRate](#type-aliases) |                                             |
| `nonce`      | [Nonce](#type-aliases)   |                                             |

Create a new [Delegation](#delegation) of `amount` coins worth of voting power for validator with address `to`.

#### SignedTransactionData: BeginUnbondingDelegation

| name         | type                     | description                                         |
| ------------ | ------------------------ | --------------------------------------------------- |
| `type`       | `TransactionType`        | Must be `TransactionType.BeginUnbondingDelegation`. |
| `maxFeeRate` | [FeeRate](#type-aliases) |                                                     |
| `nonce`      | [Nonce](#type-aliases)   |                                                     |

Begin unbonding the [Delegation](#delegation) at this address.

#### SignedTransactionData: UnbondDelegation

| name         | type                     | description                                 |
| ------------ | ------------------------ | ------------------------------------------- |
| `type`       | `TransactionType`        | Must be `TransactionType.UnbondDelegation`. |
| `maxFeeRate` | [FeeRate](#type-aliases) |                                             |
| `nonce`      | [Nonce](#type-aliases)   |                                             |

Finish unbonding the [Delegation](#delegation) at this address.

#### SignedTransactionData: Burn

| name         | type                      | description                                  |
| ------------ | ------------------------- | -------------------------------------------- |
| `type`       | `TransactionType`         | Must be `TransactionType.Burn`.              |
| `amount`     | [Amount](#type-aliases)   |                                              |
| `maxFeeRate` | [FeeRate](#type-aliases)  |                                              |
| `nonce`      | [Nonce](#type-aliases)    |                                              |
| `graffiti`   | [Graffiti](#type-aliases) | Graffiti to indicate the reason for burning. |

## IntermediateStateRootData

| name                            | type                                                              | description                               |
| ------------------------------- | ----------------------------------------------------------------- | ----------------------------------------- |
| `wrappedIntermediateStateRoots` | [WrappedIntermediateStateRoot](#wrappedintermediatestateroot)`[]` | List of wrapped intermediate state roots. |

### WrappedIntermediateStateRoot

| name                    | type                                            | description                                                                                                                                                                                                                                                                                                                  |
| ----------------------- | ----------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `index`                 | `uint64`                                        | Index of this intermediate state root in the list of intermediate state roots. This information is lost when splitting intermediate state roots into [fixed-sized shares](#share), and needs to be re-added here for fraud proof support. Allows linking an intermediate state root to a [transaction](#wrappedtransaction). |
| `intermediateStateRoot` | [IntermediateStateRoot](#intermediatestateroot) | Intermediate state root. Used for fraud proofs.                                                                                                                                                                                                                                                                              |

### IntermediateStateRoot

| name   | type                      | description                                                                              |
| ------ | ------------------------- | ---------------------------------------------------------------------------------------- |
| `root` | [HashDigest](#hashdigest) | Root of intermediate state, which is composed of the global state and the validator set. |

## EvidenceData

Wrapper for evidence data.

| name        | type                      | description                                    |
| ----------- | ------------------------- | ---------------------------------------------- |
| `evidences` | [Evidence](#evidence)`[]` | List of evidence used for slashing conditions. |

### Evidence

| name     | type                    | description |
| -------- | ----------------------- | ----------- |
| `pubKey` | [PublicKey](#publickey) |             |
| `voteA`  | [Vote](#vote)           |             |
| `voteB`  | [Vote](#vote)           |             |

### PublicKey

| name | type       | description              |
| ---- | ---------- | ------------------------ |
| `x`  | `byte[32]` | `x` value of public key. |
| `y`  | `byte[32]` | `y` value of public key. |

### Vote

```C++
enum VoteType : uint8_t {
    Prevote = 1,
    Precommit = 2,
};
```

| name               | type                    | description |
| ------------------ | ----------------------- | ----------- |
| `type`             | `VoteType`              |             |
| `height`           | [Height](#type-aliases) |             |
| `round`            | [Round](#type-aliases)  |             |
| `blockID`          | [BlockID](#blockid)     |             |
| `timestamp`        | [Timestamp](#timestamp) |             |
| `validatorAddress` | [Address](#address)     |             |
| `validatorIndex`   | `uint64`                |             |
| `signature`        | [Signature](#signature) |             |

## MessageData

| name       | type                    | description       |
| ---------- | ----------------------- | ----------------- |
| `messages` | [Message](#message)`[]` | List of messages. |

### Message

| name          | type                         | description                   |
| ------------- | ---------------------------- | ----------------------------- |
| `namespaceID` | [NamespaceID](#type-aliases) | Namespace ID of this message. |
| `rawData`     | `byte[]`                     | Raw message bytes.            |

# State

The state of the LazyLedger chain is intentionally restricted to containing only account balances and the validator set metadata. One unified [Sparse Merkle Tree](#sparse-merkle-tree) is maintained for the entire chain state, the _state tree_. The root of this tree is committed to in the [block header](#header).

The state tree is separated into `2**(8*STATE_SUBTREE_RESERVED_BYTES)` subtrees, each of which can be used to store a different component of the state. This is done by slicing off the highest `STATE_SUBTREE_RESERVED_BYTES` bytes from the key and replacing them with the appropriate [reserved state subtree ID](consensus.md#reserved-state-subtree-ids). Reducing the key size within subtrees also reduces the collision resistance of keys by `8*STATE_SUBTREE_RESERVED_BYTES` bits, but this is not an issue due the number of bits removed being small.

Three subtrees are maintained:
1. [Accounts](#account)
1. [Active validator set](#validator)
1. [Inactive validator set](#validator)

## Account

| name             | type                      | description                                                                       |
| ---------------- | ------------------------- | --------------------------------------------------------------------------------- |
| `balance`        | [Amount](#type-aliases)   | Coin balance.                                                                     |
| `nonce`          | [Nonce](#type-aliases)    | Account nonce. Every outgoing transaction from this account increments the nonce. |
| `isValidator`    | `bool`                    | Whether this account is a validator or not.                                       |
| `isDelegating`   | `bool`                    | Whether this account is delegating its stake or not.                              |
| `delegationInfo` | [Delegation](#delegation) | _Optional_, only if `isDelegating` is set. Delegation info.                       |

In the accounts subtree, accounts (i.e. leaves) are keyed by the [hash](#hashdigest) of their [address](#address). The first byte is then replaced with `ACCOUNTS_SUBTREE_ID`.

## Delegation

```C++
enum DelegationStatus : uint8_t {
    Bonded = 1,
    Unbonding = 2,
};
```

| name              | type                         | description                                         |
| ----------------- | ---------------------------- | --------------------------------------------------- |
| `status`          | `DelegationStatus`           | Status of this delegation.                          |
| `validator`       | [Address](#address)          | The validator being delegating to.                  |
| `stakedBalance`   | [VotingPower](#type-aliases) | Delegated stake, in `4u`.                           |
| `beginEntry`      | [PeriodEntry](#periodentry)  | Entry when delegation began.                        |
| `endEntry`        | [PeriodEntry](#periodentry)  | Entry when delegation ended (i.e. began unbonding). |
| `unbondingHeight` | [Height](#type-aliases)      | Block height delegation began unbonding.            |

Delegation objects represent a delegation. They have two statuses:
1. `Bonded`: This delegation is enabled for a `Queued` _or_ `Bonded` validator. Delegations to a `Queued` validator can be withdrawn immediately, while delegations for a `Bonded` validator must be unbonded first.
1. `Unbonding`: This delegation is unbonding. It will remain in this status for at least `UNBONDING_DURATION` blocks, and while unbonding may still be slashed. Once the unbonding duration has expired, the delegation can be withdrawn.

## Validator

```C++
enum ValidatorStatus : uint8_t {
    Queued = 1,
    Bonded = 2,
    Unbonding = 3,
    Unbonded = 4,
};
```

| name              | type                         | description                                                                            |
| ----------------- | ---------------------------- | -------------------------------------------------------------------------------------- |
| `status`          | `ValidatorStatus`            | Status of this validator.                                                              |
| `stakedBalance`   | [VotingPower](#type-aliases) | Validator's personal staked balance, in `4u`.                                          |
| `commissionRate`  | [Decimal](#decimal)          | Commission rate.                                                                       |
| `delegatedCount`  | `uint32`                     | Number of accounts delegating to this validator.                                       |
| `votingPower`     | [VotingPower](#type-aliases) | Total voting power as staked balance + delegated stake, in `4u`.                       |
| `pendingRewards`  | [Amount](#type-aliases)      | Rewards collected so far this period, in `1u`.                                         |
| `latestEntry`     | [PeriodEntry](#periodentry)  | Latest entry, used for calculating reward distribution.                                |
| `unbondingHeight` | [Height](#type-aliases)      | Block height validator began unbonding.                                                |
| `isSlashed`       | `bool`                       | If this validator has been slashed or not.                                             |
| `slashRate`       | [Decimal](#decimal)          | _Optional_, only if `isSlashed` is set. Rate at which this validator has been slashed. |

Validator objects represent all the information needed to be keep track of a validator. Validators have four statuses:
1. `Queued`: This validator has entered the queue to become an active validator. Once the next validator set transition occurs, if this validator has sufficient voting power (including its own stake and stake delegated to it) to be in the top `MAX_VALIDATORS` validators by voting power, it will become an active, i.e. `Bonded` validator. Until bonded, this validator can immediately exit the queue.
1. `Bonded`: This validator is active and bonded. It can propose new blocks and vote on proposed blocks. Once bonded, an active validator must go through an unbonding process until its stake can be freed.
1. `Unbonding`: This validator is in the process of unbonding, which can be voluntary (the validator decided to stop being an active validator) or forced (the validator committed a slashable offence and was kicked from the active validator set). Validators will remain in this status for at least `UNBONDING_DURATION` blocks, and while unbonding may still be slashed.
1. `Unbonded`: This validator has completed its unbonding and has withdrawn its stake. The validator object will remain in this status until `delegatedCount` reaches zero, at which point it is destroyed.

In the validators subtrees, validators are keyed by the [hash](#hashdigest) of their [address](#address). The first byte is then replaced with `ACTIVE_VALIDATORS_SUBTREE_ID` for the active validator set or `INACTIVE_VALIDATORS_SUBTREE_ID` for the inactive validator set. Active validators are `Bonded`, while inactive validators are not `Bonded`. By construction, the validators subtrees will be a subset of a mirror of the [accounts subtree](#account).

## ActiveValidatorCount

| name            | type     | description                  |
| --------------- | -------- | ---------------------------- |
| `numValidators` | `uint32` | Number of active validators. |

Since the [active validator set](#validator) is stored in a [Sparse Merkle Tree](#sparse-merkle-tree), there is no compact way of proving that the number of active validators exceeds `MAX_VALIDATORS` without keeping track of the number of active validators. The active validator count is stored in the active validators subtree, and is keyed with zero (i.e. `0x0000000000000000000000000000000000000000000000000000000000000000`), with the first byte replaced with `ACTIVE_VALIDATORS_SUBTREE_ID`.

## PeriodEntry

| name         | type                    | description                                                   |
| ------------ | ----------------------- | ------------------------------------------------------------- |
| `rewardRate` | [Amount](#type-aliases) | Rewards per unit of voting power accumulated so far, in `1u`. |

For explanation on entries, see the [reward distribution rationale document](../rationale/distributing_rewards.md).

## Decimal

TODO define a format for numbers in the range `[0,1]`

# Consensus Parameters

Various [consensus parameters](consensus.md#system-parameters) are committed to in the block header, such a limits and constants.

| name                              | type     | description                                |
| --------------------------------- | -------- | ------------------------------------------ |
| `version`                         | `uint64` | The `VERSION`.                             |
| `chainID`                         | `uint64` | The `CHAIN_ID`.                            |
| `shareSize`                       | `uint64` | The `SHARE_SIZE`.                          |
| `shareReservedBytes`              | `uint64` | The `SHARE_RESERVED_BYTES`.                |
| `availableDataOriginalSquareSize` | `uint64` | The `AVAILABLE_DATA_ORIGINAL_SQUARE_SIZE`. |

In order to compute the `consensusRoot` field in the [block header](#header), the above list of parameters is Merkleized in a plain [binary Merkle tree](#binary-merkle-tree), whose root is assigned to the `consensusRoot`.
