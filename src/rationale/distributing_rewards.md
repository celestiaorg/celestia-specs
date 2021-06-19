# Rationale: Distributing Rewards and Penalties

- [Preamble](#preamble)
- [Distribution Scheme](#distribution-scheme)
  - [State-Efficient Implementation](#state-efficient-implementation)

## Preamble

Due to the requirement that all incorrect state transitions on Celestia be provable with a [compact fraud proof](https://arxiv.org/abs/1809.09044) that is cheap enough to verify within a smart contract on a remote chain (e.g. Ethereum), computing how rewards and penalties are distributed must involve no iterations. To understand why, let us consider the following desiderata in a staking system:

1. In-protocol stake delegation: this makes it easier for users to participate in the consensus process, and reduces reliance on custodial staking services.
1. In-protocol enforcement of proper distribution of rewards and penalities to delegators: rewards and penalties collected by validators should be distributed to delegators trustlessly.

Naively, rewards and penalties (henceforth referred to collectively as "rewards", since penalties are simply negative rewards) can be distributed immediately. For example, when a validator produces a new block and is entitled to collecting transaction fees, these fees can be distributed to every single account delegating stake to this validator. This requires iterating over potentially a huge number of state elements for a single state transition (i.e. transaction), which is computationally expensive. The specific problem is that it would be infeasible to prove that such a state transition was _incorrect_ (i.e. with a fraud proof) within the execution system of a remote blockchain (i.e. with a smart contract).

This forms the primary motivation of the scheme discussed here: a mechanism for distributing rewards that is state-efficient and requires no iteration over state elements for any state transition.

## Distribution Scheme

The scheme presented here is an incarnation of Cosmos' [F1 fee distribution scheme](https://github.com/cosmos/cosmos-sdk/blob/master/docs/spec/fee_distribution/f1_fee_distr.pdf). F1 has the nice property of being approximation-free and, with proper implementation details, can be highly efficient with state usage and completely iteration-free in all cases.

Naively, when considering a single block, the reward that should be given to a delegator with stake \\( x \\), who is delegating to a validator with total voting power \\( n \\), whose reward in that block is \\( T \\), is:

$$
\text{naive reward} = x \frac{T}{n}
$$

In other words, the voting power contributed by the delegator multiplied by the _reward rate_, i.e. the rewards per unit of voting power. We note that if the total voting power of a validator remains constant forever, then the above equation holds and is approximation-free. However, changes to the total voting power need to be accounted for.

Blocks between two changes to a validator's voting power (i.e. whenever a user delegates or undelegates stake) are a _period_. Every time a validator's voting power changes (i.e. a new period \\( f \\) begins), an entry \\( Entry_f \\) for this period is saved in state, which records _the reward rate up to the beginning of_ \\( f \\). This is simply the sum of the reward rate up to the beginning of previous period \\( f-1 \\) and the reward rate of the period \\( f \\) itself:

$$
Entry_f = \begin{cases}
    0 & f = 0 \\\\
    Entry_{f-1} + \frac{T_f}{n_f} & f > 0 \\\\
\end{cases}
$$

Note that \\( Entry \\) is a monotonically increasing function.

Finally, the raw reward for a delegation is simply proportional to the difference in entries between the period where undelegation ended (\\( f \\)) and where it began (\\( k \\)).

$$
\text{reward} = x (Entry_f - Entry_k)
$$

This raw reward can be scaled by additional factors, such as commissions or slashing penalties.

### State-Efficient Implementation

The F1 paper does not specify where entries are stored in state, but the understanding is that they are placed in independent state elements. This has the downside of requiring multiple Merkle branches to prove the inclusion of entries for e.g. fraud proofs. We can improve on this by leveraging a specific property of entries, namely that each entry is only used in exactly two cases:

1. To compute the next entry.
1. To compute the reward of a delegator.

Intuitively, after having being used twice, an entry can be pruned from the state. We can make use of this by storing only the latest entry with its respective validator object, and a copy of the two entries each delegation needs with the delegation object. By storing entries directly with the objects that require them, state transitions can be statelessly validated without extra inclusion proofs.
