Architecture
===

# Common Terms and Expressions

| name              | description                                                                          |
| ----------------- | ------------------------------------------------------------------------------------ |
| App (application) | Alternate name for "virtual sidechain."                                              |
| Transaction       | Requests that modify the consensus-critical state (validator balances and statuses). |
| Message           | Requests that are executed by non-consensus-critical apps.                           |

# Proposal 1: Validator Set in State, Fees in App

In this proposal, LazyLedger has a minimal state: the validator set. Changes to the validator set are done with native _transactions_, distinct from the _messages_ processed by apps. Transactions pay fees just as they would in a normal blockchain (e.g. Bitcoin), and their side effects are essentially restricted to modifying the validator set and their balances. The validator set is committed to in the block header.

Fees for including messages are paid for within a non-consensus-critical app, which has a completely separate balance sheet to the validator set state. All block proposers are expected to, in practice, use a single app to accept and guarantee proper fee payments for the work they do to include messages in blocks, but this app is not consensus critical and so non-consensus nodes do not need to run it.

Pros:
* Non-consensus full clients have easy and direct access to all the data they need to validate: the transactions and the validator set changed by those transactions. No extraneous data needs to be picked out, as fees are segregated to an app.
* Light clients are identical to full clients in this proposal: they simply need to process all the validator set changes (i.e. transactions) and run data availability checks.

Cons:
* Compared to the other proposals, additional commitments are needed in the block header.
* Due to having two separate environments for validators and fee payments, and LazyLedger not supporting execution at the base layer, only a one-way bridge validator set $\rightarrow$ fee payment is allowed. Transfer to funds backwards can be simulated to a limited extent using atomic swaps between the two environments---HTLCs are pure with respect to state and supporting then with transactions should not add meaningful overhead. Tax and market (i.e. potentially two coins) considerations might come into play.

# Proposal 2: Validator Set in a Consensus-Critical App, Fees in App

As above, but the validator set is processed in a consensus-critical app. In other words, commitments to the validator set aren't in the LazyLedger block header, but in the app's header.

Pros:
* Fewer commitments in block headers.
* Potentially cleaner design, with no pollution in the base layer.

Cons:
* Requires both non-consensus full nodes and light nodes to additionally download and process Merkle paths to the consensus-critical app's header.
* As above, only a one-way bridge is supported.

# Proposal 3: Validator Set and Fees in a Consensus-Critical App

In this proposal, both the validator set changes and fee payments are put into a consensus-critical app.

Pros:
* Only a single coin for everything. No bridging between the two systems is needed.

Cons:
* Both full and light clients must either execute fee payments alongside the validator set changes, or at least download and parse but ignore them.
