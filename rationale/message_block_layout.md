Message Layout
===

- [Message Layout](#message-layout)
- [Preamble](#preamble)
- [Message Layout Rationale](#message-layout-rationale)

# Preamble

LazyLedger uses [a data availability scheme](https://arxiv.org/abs/1809.09044) that allows nodes to determine whether a block's data was published without downloading the whole block. The core of this scheme is arranging data in a two-dimensional matrix then applying erasure coding to each row and column. This document describes the rationale for how data—transactions, messages, and other data—[is actually arranged](../specs/data_structures.md#arranging-available-data-into-shares). Familiarity with the [originally proposed data layout format](https://arxiv.org/abs/1809.09044) is assumed.

# Message Layout Rationale

Block data consists of transactions (which modify the LazyLedger chain's state), intermediate state roots (required for fraud proofs of the aforementioned transactions), messages (binary blobs which do not modify the LazyLedger state, but which are intended for a LazyLedger application identified with a provided namespace ID), and other relevant pieces of data (e.g. evidence for slashing). We want to arrange this data into a `k * k` matrix of fixed-sized shares, which will later be committed to in [Namespace Merkle Trees (NMTs)](../specs/data_structures.md#namespace-merkle-tree).

The simplest way we can imagine arranging block data is to simply serialize it all in no particular order, split it into fixed-sized shares, then arrange those shares into the `k * k` matrix in row-major order. However, this naive scheme can be improved in a number of ways, described below.

First, we impose some ground rules:
1. Data must be ordered by namespace ID. This makes queries into a NMT commitment of that data more efficient.
1. Since non-message data are not naturally intended for particular namespaces, we assign reserved namespaces for them. A range of namespaces is reserved for this purpose, starting from the lowest possible namespace ID.
1. By construction, the above two rules mean that non-message data always precedes message data in the row-major matrix, even when considering single rows or columns.
1. Data with different namespaces must not be in the same share. This might cause a small amount of wasted block space, but makes the NMT easier to reason about in general since leaves are guaranteed to belong to a single namespace, which will be an important property shorty.

Transactions can pay fees for the inclusion of a particular message. However, we do not want transactions to include the entire message they pay for (which is the case in other blockchains with native execution, e.g. calldata in Ethereum transactions or OP_RETURN data in Bitcoin transactions), otherwise every node that validates the sanctity of the LazyLedger coin would need to download all message data. Transactions must therefore only include a commitment to the message they pay fees for, which can lead to a data availability problem if implemented naively (e.g. with a simple hash of the message, or a simple binary Merkle tree root of the message).

To that end, we impose some additional rules onto _messages only_: messages must be placed is a way such that both the transaction sender and the block producer can be held accountable—a necessary property for e.g. fee burning. Accountable in this context means that
1. The transaction sender must pay sufficient fees for message inclusion.
1. The block proposer cannot claim that a message was included when it was not (which implies that a transaction and the message it pays for must be included in the same block).

We note a nice property: if the transaction sender knows 1) `k`, the size of the matrix, 2) the starting location of their message in a row, and 3) the length of the message (they know this since they are sending the message), then they can actually compute a sequence of roots to _subtrees in the row NMTs_. More importantly, anyone can compute this, and can compute _the simple Merkle root of these subtree roots_.

All transaction senders knowing the exact starting location of their message is unrealistic however, as that would require interaction between the block producer and transaction senders. We can impose some additional rules on message placement to make the possible starting locations of messages sufficiently predictable and constrained such that users can deterministically compute subtree roots without interaction:
1. Messages begin at a new share, unlike non-message data which can span multiple shares.
1. Messages that span multiple rows must begin at the start of a row (this can occur if a message is longer than `k` shares _or_ if the block producer decides to start a message partway through a row and it cannot fit).
1. Messages begin at a location aligned with the largest power of 2 that is not larger than the message length or `k`.

With the above constraints, we can compute subtree roots deterministically easily: simply slice off either the largest power of 2 that isn't larger than the remaining message length, or `k`, whichever is smaller. This is done recursively. As an example, with `k = 4` and message length of `11`, the message would be sliced with lengths `4, 4, 2, 1`. The resulting slices are the leaves of subtrees whose roots can be computed. Due to the rules above, the subtrees are guaranteed to be aligned to powers of 2, and thus the subtree roots will be present as internal nodes in the NMT of _some_ row(s).

This is similar to [Merkle Mountain Ranges](https://www.usenix.org/legacy/event/sec09/tech/full_papers/crosby.pdf), though with the largest subtree bounded by `k` rather than being unbounded.

The last piece of the puzzle is determining _which_ row the message is placed at (or, more specifically, the starting location). This is needed to keep the block producer accountable. To this end, the block producer simply augments each fee-paying transaction with some metadata: the starting location of the message the transaction pays for.
