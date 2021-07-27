# Networking

- [Wire Format](#wire-format)
  - [AvailableData](#availabledata)
  - [AvailableDataRow](#availabledatarow)
  - [ConsensusProposal](#consensusproposal)
  - [WireTxPayForMessage](#wiretxpayformessage)
- [Invalid Erasure Coding](#invalid-erasure-coding)
  - [ShareProof](#shareproof)
  - [BadEncodingFraudProof](#badencodingfraudproof)
- [Invalid State Update](#invalid-state-update)
  - [StateFraudProof](#statefraudproof)

## Wire Format

### AvailableData

| name                | type                                      | description   |
|---------------------|-------------------------------------------|---------------|
| `availableDataRows` | [AvailableDataRow](#availabledatarow)`[]` | List of rows. |

### AvailableDataRow

| name     | type                                    | description      |
|----------|-----------------------------------------|------------------|
| `shares` | [Share](./data_structures.md#share)`[]` | Shares in a row. |

### ConsensusProposal

Defined as `ConsensusProposal`:

```protobuf
{{#include ./proto/consensus.proto:ConsensusProposal}}
```

When receiving a new block proposal `proposal` from the network, the following steps are performed in order. _Must_ indicates that peers must be blacklisted (to prevent DoS attacks) and _should_ indicates that the network message can simply be ignored.

1. `proposal.type` must be a `SignedMsgType`.
1. `proposal.round` is processed identically to Tendermint.
1. `proposal.pol_round` is processed identically to Tendermint.
1. `proposal.header` must be well-formed.
1. `proposal.header.version.block` must be [`VERSION_BLOCK`](./consensus.md#constants).
1. `proposal.header.version.app` must be [`VERSION_APP`](./consensus.md#constants).
1. `proposal.header.height` should be previous known height + 1.
1. `proposal.header.chain_id` must be [`CHAIN_ID`](./consensus.md#constants).
1. `proposal.header.time` is processed identically to Tendermint.
1. `proposal.header.last_header_hash` must be previous block's header hash.
1. `proposal.header.last_commit_hash` must be the previous block's commit hash.
1. `proposal.header.consensus_hash` must be the hash of [consensus parameters](./data_structures.md#header).
1. `proposal.header.state_commitment` must be the state root after applying the previous block's transactions.
1. `proposal.header.available_data_original_shares_used` must be at most [`AVAILABLE_DATA_ORIGINAL_SQUARE_MAX ** 2`](./consensus.md#constants).
1. `proposal.header.available_data_root` must be the [root](./data_structures.md#availabledataheader) of `proposal.da_header`.
1. `proposal.header.proposer_address` must be the [correct leader](./consensus.md#leader-selection).
1. `proposal.da_header` must be well-formed.
1. The number of elements in `proposal.da_header.row_roots` and `proposal.da_header.row_roots` must be equal.
1. The number of elements in `proposal.da_header.row_roots` must be the same as computed [here](./data_structures.md#header).
1. `proposal.proposer_signature` must be a valid [digital signature](./data_structures.md#public-key-cryptography) over the header hash of `proposal.header` that recovers to `proposal.header.proposer_address`.
1. For [full nodes](./node_types.md#node-type-definitions), `proposal.da_header` must be the result of computing the roots of the shares (received separately).
1. For [light nodes](./node_types.md#node-type-definitions), `proposal.da_header` should be sampled from for availability.

### WireTxPayForMessage

Defined as `WireTxPayForMessage`:

```protobuf
{{#include ./proto/wire.proto:WireTxPayForMessage}}
```

Accepting a `WireTxPayForMessage` into the mempool requires different logic than other transactions in Celestia, since it leverages the paradigm of block proposers being able to malleate transaction data. Unlike [SignedTransactionDataPayForMessage](./data_structures.md#signedtransactiondatapayformessage) (the canonical data type that is included in blocks and committed to with a data root in the block header), each `WireTxPayForMessage` (the over-the-wire representation of the same) has potentially multiple signatures.

Transaction senders who want to pay for a message will create a [SignedTransactionDataPayForMessage](./data_structures.md#signedtransactiondatapayformessage) object, `stx`, filling in the `stx.messageShareCommitment` field [based on the non-interactive default rules](../rationale/message_block_layout.md#non-interactive-default-rules) for `k = AVAILABLE_DATA_ORIGINAL_SQUARE_MAX`, then signing it to get a [transaction](./data_structures.md#transaction) `tx`. This process is repeated with successively smaller `k`s, decreasing by powers of 2 until `k * k <= stx.messageSize`. At that point, there would be insufficient shares to include both the message and transaction. Using the rest of the signed transaction data along with the pairs of `(tx.signedTransactionData.messageShareCommitment, tx.signature)`, a `WireTxPayForMessage` object is constructed.

Receiving a `WireTxPayForMessage` object from the network follows the reverse process: for each `message_commitment_and_signature`, verify using the [based on the non-interactive default rules](../rationale/message_block_layout.md#non-interactive-default-rules) that the signature is valid.

## Invalid Erasure Coding

If a malicious block producer incorrectly computes the 2D Reed-Solomon code for a block's data, a fraud proof for this can be presented.

### ShareProof

| name       | type                                                  | description                                                                                       |
|------------|-------------------------------------------------------|---------------------------------------------------------------------------------------------------|
| `share`    | [Share](#share)                                       | The share.                                                                                        |
| `proof`    | [NamespaceMerkleTreeProof](#namespacemerkletreeproof) | The Merkle proof of the share in [`availableDataRoot`](#header).                                  |
| `isCol`    | `bool`                                                | A Boolean indicating if the proof is from a row root or column root; `false` if it is a row root. |
| `position` | `uint64`                                              | The index of the share in the offending row or column.                                            |

### BadEncodingFraudProof

| name          | type                                                                                  | description                                                                                           |
|---------------|---------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------|
| `shareProofs` | [ShareProof](./data_structures.md#shareproof)`[]`                                     | The available shares in the offending row or column.                                                 |
| `root`        | [HashDigest](./data_structures.md#hashdigest)                                         | The Merkle root of the offending row or column.                                                       |
| `proof`       | [BinaryMerkleTreeInclusionProof](./data_structures.md#binarymerkletreeinclusionproof) | The Merkle proof of the row or column root in [`availableDataRoot`](./data_structures.md#header).|
| `isCol`       | `bool`                                                                                | A Boolean indicating if it is an offending row or column; `false` if it is a row.                     |
| `position`    | `uint64`                                                                              | The index of the row or column in the square.                                                         |
## Invalid State Update

If a malicious block producer incorrectly computes the state, a fraud proof for this can be presented.

### StateFraudProof

| name                       | type                                                                                     | description                                                                                                                                                                                            |
|----------------------------|------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `transactionShareProofs`   | [ShareProof](#shareproof)`[]`                                                            | `isCol` of type `bool` must be `false`.                                                                                                                                                                |
| `stateShareProofs`         | [ShareProof](#shareproof)`[]`                                                            | `isCol` of type `bool` must be `false`.                                                                                                                                                                |
| `index`                    | `uint64`                                                                                 | Index for connecting the [WrappedIntermediateStateRoot](./data_structures.md#wrappedintermediatestateroot) and [WrappedTransaction](./data_structures.md#wrappedtransaction) after shares are parsed. |
| `intermediateStateElements`| [StateElement](./data_structures.md#stateelement)`[]`                                   | State elements that were changed by the transactions.                                                                                                                                                  |
| `stateInclusionProofs`     | [SparseMerkleTreeInclusionProof](./data_structures.md#sparsemerkletreeinclusionproof)`[]`| SparseMerkleTree inclusion proofs for the state elements.                                                                                                                                      |
