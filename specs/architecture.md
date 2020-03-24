Architecture
===

- [Architecture](#architecture)
- [Common Terms and Expressions](#common-terms-and-expressions)
- [System Architecture](#system-architecture)
- [Proposal 1: Validator Set in State, Fees in App](#proposal-1-validator-set-in-state-fees-in-app)
- [Proposal 2: Validator Set and Fee Payments in State](#proposal-2-validator-set-and-fee-payments-in-state)
- [Proposal 3: Validator Set in a Consensus-Critical App, Fees in App](#proposal-3-validator-set-in-a-consensus-critical-app-fees-in-app)
- [Proposal 4: Validator Set and Fees in a Consensus-Critical App](#proposal-4-validator-set-and-fees-in-a-consensus-critical-app)
- [Proposal 5: Fixed Burn For Fees](#proposal-5-fixed-burn-for-fees)

# Common Terms and Expressions

| name              | description                                                                                 |
| ----------------- | ------------------------------------------------------------------------------------------- |
| App (application) | Alternate name for "[virtual sidechain](https://arxiv.org/abs/1905.09274)." LazyLedger apps |
| Transaction       | Requests that modify the consensus-critical state (validator balances and statuses).        |
| Message           | Requests that are executed by non-consensus-critical apps.                                  |


# System Architecture

![fig: Block data structures.](figures/block_data_structures.svg)

# Proposal 1: Validator Set in State, Fees in App

In this proposal, LazyLedger has a minimal state: the validator set. Changes to the validator set are done with native _transactions_, distinct from the _messages_ processed by apps. Transactions pay fees just as they would in a normal blockchain (e.g. Bitcoin), and their side effects are essentially restricted to modifying the validator set and their balances. The validator set is committed to in the block header.

Fees for including messages are paid for within a non-consensus-critical app, which has a completely separate balance sheet to the validator set state. All block proposers are expected to, in practice, use a single app to accept and guarantee proper fee payments for the work they do to include messages in blocks, but this app is not consensus critical and so non-consensus nodes do not need to run it.

Pros:
* Non-consensus full clients have easy and direct access to all the data they need to validate: the transactions and the validator set changed by those transactions. No extraneous data needs to be picked out, as fees are segregated to an app.
* Light clients are almost identical to full clients in this proposal: they simply need to process all the validator set changes (i.e. transactions) and run data availability checks. Note that unlike full client, light clients do not need to verify the signatures of transactions and can instead trust the majority of validators to sign off on validator set changes, with the addition of fraud proofs in case of an invalid signature.

Cons:
* Compared to the other proposals, additional commitments are needed in the block header.
* Due to having two separate environments for validators and fee payments, and LazyLedger not supporting execution at the base layer, only a one-way bridge validator set $\rightarrow$ fee payment is allowed. Transfer to funds backwards can be simulated to a limited extent using atomic swaps between the two environments---HTLCs are pure with respect to state and supporting then with transactions should not add meaningful overhead. Tax and market (i.e. potentially two coins) considerations might come into play.

# Proposal 2: Validator Set and Fee Payments in State

As above, but both the validator set changes and fee payments are transactions that modify state.

Pros:
* Single coin, no need to worry about bridging.
* This can be built entirely using the Cosmos SDK for Tendermint apps.

Cons:
* More work for full clients to validate transactions.

# Proposal 3: Validator Set in a Consensus-Critical App, Fees in App

As in proposal 1, but the validator set is processed in a consensus-critical app. In other words, commitments to the validator set aren't in the LazyLedger block header, but in the app's header.

Pros:
* Fewer commitments in block headers.
* Potentially cleaner design, with no pollution in the base layer.

Cons:
* Requires both non-consensus full nodes and light nodes to additionally download and process Merkle paths to the consensus-critical app's header.
* As above, only a one-way bridge is supported.

# Proposal 4: Validator Set and Fees in a Consensus-Critical App

In this proposal, both the validator set changes and fee payments are put into a consensus-critical app.

Pros:
* Only a single coin for everything. No bridging between the two systems is needed.

Cons:
* Both full and light clients must either execute fee payments alongside the validator set changes, or at least download and parse but ignore them.

# Proposal 5: Fixed Burn For Fees

Some fixed amount must be paid for including a message. This amount is burned, not given to the block proposer. Since this is consensus-critical, only proposals 2 and 4 above can be extended to include this feature.

Pros:
* This provides baseline demand for the native coin: as the chain is used more, more coins must be bought then burned to pay for fees.
* No other big chain has incorporated such a mechanism, but several have it in the pipeline to investigate.

Cons:
* Setting the amount to burn is non-trivial. Too low and it won't be meaningful. Too high and it will be too expensive if the price of the coin rises. The amount must be flexible in some way.
