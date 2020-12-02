Fees
===

- [Preamble](#preamble)
- [Background: EIP-1559](#background-eip-1559)
- [Fee Burning](#fee-burning)

## Preamble

Two desiderata are elastic block size limits—which can smooth out transaction fees and prevent fee spikes—and variable burning of fees—which can respond to market forces and provides intrinsic demand for the burned coin that cannot be replicated by issuance of new supply. Interestingly, these two properties cannot be accomplished independently. Without a negative feedback mechanism that cannot be bypassed, block producers will always produce maximum-size blocks. This negative feedback mechanism is in-protocol variable fee burning. In a similar vein, variable (rather than fixed) fee burning requires a measure of block usage, which happens to exactly be an elastic block size limit.

This document describes the rationale and structure of an elastic block size limit through variable in-protocol fee burning.

## Background: EIP-1559

[EIP-1559](https://eips.ethereum.org/EIPS/eip-1559) introduces a concrete proposal for in-protocol variable fee burning for Ethereum, with only a superficial analysis of properties. [This paper](http://timroughgarden.org/papers/eip1559.pdf) provides a rigorous formal analysis of the same.

The essence of EIP-1559 is that, rather than the [first-price auction fee rate](https://arxiv.org/abs/1901.06830) of traditional cryptocurrencies, transactions under EIP-1559 define two fee rate parameters: a maximum base fee rate and a tip rate. The base fee is burned as a mechanism for enabling an elastic block size, and the tip is given to the miner of the block that includes the transaction (and is itself subject to a first-price auction). All transactions in a block burn the same base fee.

The base fee grows if the previous block was more than half-full, and shrinks if the previous block was less than half full, by a parameter. We note, as above, that fee burning is simply a mechanism for making an elastic block size limit viable. This parameter should be chosen with the following in mind:
1. The target (or, expected) block size should be bounded by the cost to run a full node assuming all blocks are at the target block size (also known as "decentralization").
1. The maximum block size should bounded by the size of a "poison" block, i.e. one that would disrupt the consensus protocol.

From this, we see that there is nothing inherent in half-fullness, and indeed this parameter should be chosen more carefully through empirical results.

## Fee Burning

Since blocks grow in size by a power of 4 due to the arrangement of data in a square, a more reasonable initial choice of parameters would be 1:4 target:maximum block size limits. We can then define the base fee as

$$
b_{cur} = b_{prev} \cdot \left( 1 + R \cdot \frac{s_{prev} - S_{target}}{S_{target}} \right)
$$

, where $b_{cur}$ is the base fee of the current block, $b_{prev}$ is the base fee of the previous block, $R$ is the rate of change in base fee, $s_{prev}$ is the size of the previous block, and $S_{target}$ is the target block size. In other words, the base fee is only dependent on the current and previous blocks.

As an example, if $R = \frac{1}{8}$ (the rate defined by EIP-1559), the base fee can reduce by up to $12.5\%$ if the current block is completely empty and increase by up to $37.5\%$ if the current block is full.
