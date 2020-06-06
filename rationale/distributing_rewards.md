Rationale: Distributing Rewards and Penalties
===

- [Rationale: Distributing Rewards and Penalties](#rationale-distributing-rewards-and-penalties)
- [Distributing Rewards and Penalties](#distributing-rewards-and-penalties)

# Distributing Rewards and Penalties

Due to the requirement that all incorrect state transitions be provable with a compact fraud proof that is cheap enough to verify within a smart contract on a remote chain, computing rewards and penalties must involve minimal or no iterations. The scheme presented here is inspired by Cosmos' [F1 fee distribution scheme](https://github.com/cosmos/cosmos-sdk/blob/master/docs/spec/_proposals/f1-fee-distribution/f1_fee_distr.pdf) and the concept of "[coin days](https://bitcointalk.org/index.php?topic=6172.msg90789#msg90789)."

F1 requires iterating over (potentially) a large number of blocks, but avoids needing to iterate over every delegation, all while being approximation-free. As such, it cannot be used for LazyLedger directly. The scheme presented here requires no iterations at all, but is not approximation-free. The intuition is that rewards of a delegation (or validator) are simply proportional to the accumulated voting power contributed by that delegation (or validator's stake) and the remaining accumulated voting power. This has the effect of "smoothing out" rewards, hence the lack of approximation-freeness.

