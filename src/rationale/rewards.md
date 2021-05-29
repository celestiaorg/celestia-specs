# Block Rewards

Block rewards scale with the inverse square root of the total validating stake. This gives us a nice property: as the total validating stake decreases, returns per validator increases. This encourages additional validators to join and makes the system as a whole more robust even in the presence of secondary uses of the staking coin, e.g. being used as collateral in Decentralized Finance protocols.

Note that non-constant reward scaling opens up the system to [gatekeeping attacks](https://arxiv.org/abs/1811.00742), whereby validators are incentivized to prevent new validators from joining the validator set to keep their returns high. This should not be an issue in practice in the same way as [feather forks](https://bitcointalk.org/index.php?topic=312668.0) are not an issue in practice, but is nonetheless a theoretical issue that is noted here.

The formula to calculate the reward per block uses the following symbols:

| symbol      | note                              |
|-------------|-----------------------------------|
| \\( R_B \\) | Rewards per block, in coins.      |
| \\( I_T \\) | Target annual issuance, in coins. |
| \\( t_B \\) | Block time, in seconds.           |
| \\( t_Y \\) | Seconds per year.                 |
| \\( S_0 \\) | Initial coin supply.              |
| \\( S_T \\) | Total staked coins.               |

Note that for the seconds per year we use a fixed `31,536,000`, omitting leap seconds for simplicity.

The reward for a given block is thus only dependent on the validating stake, with remaining terms being constant:

$$
R_B(S_T) = I_T \frac{t_B}{t_Y} \frac{\sqrt{S_T}}{\sqrt{S_0}} = \left( \frac{I_T t_B }{t_Y \sqrt{S_0}} \right) \sqrt{S_T}
$$

If 100% of the initial supply of coins stake, then exactly \\( I_T \\) new coins will be issued per \\( \frac{t_Y}{t_B} \\) blocks. If fewer than 100% of the initial supply of coins stake, then fewer than \\( I_T \\) new coins will be issued, but the decrease will be square-root rather than linear. If greater than 100% of the initial supply of coins stake (e.g. after some time, additional coins will be in circulation due to previous issuance), then greater than \\( I_T \\) new coins will be issued, but again scaling sub-linearly.
