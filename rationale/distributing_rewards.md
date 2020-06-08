Rationale: Distributing Rewards and Penalties
===

- [Rationale: Distributing Rewards and Penalties](#rationale-distributing-rewards-and-penalties)
- [Preamble](#preamble)
- [Background](#background)
- [Distribution Scheme](#distribution-scheme)

# Preamble

Due to the requirement that all incorrect state transitions on LazyLedger be provable with a [compact fraud proof](https://arxiv.org/abs/1809.09044) that is cheap enough to verify within a smart contract on a remote chain (e.g. Ethereum), computing how rewards and penalties are distributed must involve minimal or (ideally) no iterations. To understand why, let us consider the following desiderata in a staking system:
1. In-protocol stake delegation: this makes it easier for users to participate in the consensus process, and reduces reliance on custodial staking services.
1. In-protocol enforcement of proper distribution of rewards and penalities to delegators: rewards and penalties collected by validators should be distributed to delegators trustlessly.

Naively, rewards and penalties (henceforth referred to collectively as "rewards", since penalties are simply negative rewards) can be distributed immediately. For example, when a validator produces a new block and is entitled to collecting transaction fees, these fees can be distributed to every single account delegating stake to this validator. This requires iterating over potentially a huge number of state elements for a single state transition (i.e. transaction), which is computationally expensive. The specific problem is that it would be infeasible to prove that such a state transition was _incorrect_ (i.e. with a fraud proof) within the execution system of a remote blockchain (i.e. with a smart contract).

This forms the primary motivation of the reward distribution scheme presented in this document: a mechanism for distributing rewards that is "good enough" while requiring no iteration over state elements for any state transition.

# Background

F1 requires iterating over (potentially) a large number of blocks, but avoids needing to iterate over every delegation, all while being approximation-free. As such, it cannot be used for LazyLedger directly.

# Distribution Scheme

 The scheme presented here is inspired by Cosmos' [F1 fee distribution scheme](https://github.com/cosmos/cosmos-sdk/blob/master/docs/spec/_proposals/f1-fee-distribution/f1_fee_distr.pdf) and the concept of "[coin days](https://bitcointalk.org/index.php?topic=6172.msg90789#msg90789)."

 The scheme presented here requires no iterations at all, but is not approximation-free. The intuition is that rewards of a delegation (or validator) are simply proportional to the accumulated voting power contributed by that delegation (or validator's stake) and the remaining accumulated voting power. This has the effect of "smoothing out" rewards, hence the lack of approximation-freeness.

Every time a bonded validator's voting power changes (i.e. when a delegation is added or removed), or when a validator begins unbonding, the rate at which accumulated voting power grows also changes. Intuitively, this "accumulated voting power" is similar to "coin days," but measures voting power over a number of blocks instead of coins over a number of days. The height of the last time the voting power of this validator was changed is updated to the current block height and the accumulated voting power is increased.
