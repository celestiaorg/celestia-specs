# Node Types

- [Node Parameters](#node-parameters)
  - [Block Headers](#block-headers)
    - [Compact Block Headers](#compact-block-headers)
    - [Extended Block Headers](#extended-block-headers)
  - [Block Bodies](#block-bodies)
    - [No Bodies](#no-bodies)
    - [Sampled Bodies](#sampled-bodies)
    - [Partial Bodies](#partial-bodies)
    - [Full Bodies](#full-bodies)
  - [Transactions](#transactions)
    - [No Transactions](#no-transactions)
    - [Full Transactions](#full-transactions)
- [Node Type Definitions](#node-type-definitions)

## Node Parameters

Nodes that run the [Celestia protocol](./consensus.md) have a number of parameters that can be tweaked with regards to which parts of the data is downloaded, validated, and/or stored. All nodes process the `header` and `lastCommit` fields of each [block](./data_structures.md#block), but can handle the `availableDataHeader` and `availableData` fields differently.

We define security assumptions as assumptions under which a given node is guaranteed accountable consensus safety (i.e. that finalized blocks will remain in the prefix of all future sequences of blocks accepted by the node, unless a supermajority (> 2/3) of validator voting power performs an attributable—and thus [penalizable](./consensus.md#blockavailabledataevidencedata)—malicious action) and state safety (i.e. that an invalid state transition will not be included in the chain accepted by the node).

### Block Headers

#### Compact Block Headers

Nodes that only process compact block headers will download and validate the [block header](./data_structures.md#header) _without_ downloading or validating the `availableDataHeader` [block](./data_structures.md#block) field that is committed to in the block header. These nodes cannot perform Data Availability Sampling on [block bodies](#block-bodies).

Secure under an honest supermajority of validator voting power and a weak subjectivity assumption.

#### Extended Block Headers

Nodes that process extended block headers will download and validate both the [compact block header](#compact-block-headers) and the
[`availabledataheader`](./data_structures.md##availabledataheader) [block](./data_structures.md#block) field. These nodes can perform Data Availability Sampling on [block bodies](#block-bodies), and their security assumptions depend on how block bodies are handled.

### Block Bodies

Block bodies (the `availableData` [block](./data_structures.md#block) field) can be downloaded and optionally stored and/or served. Storing and serving block body data has no effect on node security assumptions.

#### No Bodies

Nodes that only process [compact block headers](#compact-block-headers) have no need for block bodies and simply do not process block bodies.

Secure under an honest supermajority majority of validator voting power and a weak subjectivity assumption.

#### Sampled Bodies

These nodes perform Data Availability Sampling on block bodies.

Secure under an honest minority of nodes and a weak subjectivity assumption.

#### Partial Bodies

These nodes fully download and validate [the erasure coding](./data_structures.md#2d-reed-solomon-encoding-scheme) of a random subset of block bodies (configurable locally). Since the erasure coding of each block is stateless, nodes that perform validation of partial bodies contribute to the overall security of the network by being able to produce [fraud proofs of invalid erasure coding](./data_structures.md#invalid-erasure-coding).

Secure under an honest minority of nodes and a weak subjectivity assumption.

#### Full Bodies

These nodes fully download and validate [the erasure coding](./data_structures.md#2d-reed-solomon-encoding-scheme) of all block bodies.

If [transactions are not processed](#no-transactions), secure under an honest minority of nodes and a weak subjectivity assumption. If [transactions are processed](#full-transactions), secure under a weak subjectivity assumption.

### Transactions

#### No Transactions

These nodes process do not process requests [with a reserved namespace ID](./data_structures.md#arranging-available-data-into-shares) and thus to not know the chain state without relying on a third party.

At most secure under an honest minority of nodes and a weak subjectivity assumption.

#### Full Transactions

Nodes that wish to produce new blocks must know the [chain state](./data_structures.md#state). Processing all block bodies is actually not needed to know the Celestia state, as [transactions that pay for message inclusion commit to messages](../rationale/message_block_layout.md). These nodes process all requests [with a reserved namespace ID](./data_structures.md#arranging-available-data-into-shares) from block bodies and perform Data Availability Sampling for the remaining (message) data.

At most secure under a weak subjectivity assumption.

## Node Type Definitions

For convenience, we will define several common parameter configurations:

1. [Full nodes](https://en.bitcoin.it/wiki/Full_node) provide the strongest security guarantees.
    - Block headers: [Extended Block Headers](#extended-block-headers)
    - Block bodies: [Full Bodies](#full-bodies)
        - Storage: Not stored beyond a parameter
        - Serving: Served to the network
    - Transactions: [Full Transactions](#full-transactions)
    - Fraud proofs:
        - State transition: produce, consume
        - Erasure coding: produce, consume
1. Partial nodes are capable of producing fraud proofs of invalid transactions and contribute to validating the erasure coding of random blocks.
    - Block headers: [Extended Block Headers](#extended-block-headers)
    - Block bodies: [Partial Bodies](#partial-bodies)
        - Storage: Not stored beyond a parameter
        - Serving: Served to the network
    - Transactions: [Full Transactions](#full-transactions)
    - Fraud proofs:
        - State transition: produce, consume
        - Erasure coding: produce (partial), consume
1. Light nodes perform Data Availability Sampling (DAS) and are secure under an honest minority.
    - Block headers: [Extended Block Headers](#extended-block-headers)
    - Block bodies: [Sampled Bodies](#sampled-bodies)
        - Storage: Not stored beyond a parameter
        - Serving: Served to the network
    - Transactions: [No Transactions](#no-transactions)
    - Fraud proofs:
        - State transition: consume
        - Erasure coding: consume
1. Superlight nodes do not perform DAS and are secure under an honest majority.
    - Block headers: [Compact Block Headers](#compact-block-headers)
    - Block bodies: [No Bodies](#no-bodies)
        - Storage: Not stored
        - Serving: Not served to the network
    - Transactions: [No Transactions](#no-transactions)
    - Fraud proofs:
        - State transition: consume
        - Erasure coding: consume
1. Storage nodes provide the same security guarantees as full nodes.
    - Block headers: [Extended Block Headers](#extended-block-headers)
    - Block bodies: [Full Bodies](#full-bodies)
        - Storage: Fully stored in [erasure-coded form](./data_structures.md#2d-reed-solomon-encoding-scheme)
        - Serving: Served to the network
    - Transactions: [Full Transactions](#full-transactions)
    - Fraud proofs:
        - State transition: produce, consume
        - Erasure coding: produce, consume
1. Partial storage nodes provide the same security guarantees as partial nodes.
    - Block headers: [Extended Block Headers](#extended-block-headers)
    - Block bodies: [Full Bodies](#full-bodies)
        - Storage: Partially stored in [erasure-coded form](./data_structures.md#2d-reed-solomon-encoding-scheme)
        - Serving: Served to the network
    - Transactions: [Full Transactions](#full-transactions)
    - Fraud proofs:
        - State transition: produce, consume
        - Erasure coding: produce (partial), consume
1. Validator nodes are identical to full nodes, but should additionally create and sign consensus messages (proposals, votes, etc.).
    - Block headers: [Extended Block Headers](#extended-block-headers)
    - Block bodies: [Full Bodies](#full-bodies)
        - Storage: Not stored beyond a parameter
        - Serving: Served to the network
    - Transactions: [Full Transactions](#full-transactions)
    - Fraud proofs:
        - State transition: produce, consume
        - Erasure coding: produce, consume
