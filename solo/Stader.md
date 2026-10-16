# Stader Labs, participated in Code4rena

*********************review commit hash -********************* - **[a46e9e439cd8664e06f942050857bfe4a776c04a](https://github.com/code-423n4/2023-06-stader-findings)**

# Issue M-1: a validator's SD collateral would be entirely slashed even if it's collateral is slightly less than the penalty

Judged as a duplicate of [M-06] Protocol will not benefit from slashing mechanism when remaining penalty bigger than `minThreshold` in the [final report](https://code4rena.com/reports/2023-06-stader).

[Link to the original submission](https://github.com/code-423n4/2022-06-stader-findings/issues/68)


# Issue M-2: no bidder has incentive to bid the Auction except doing last-minute MEV due to fixed endBlock

Selected as the primary report for [M-13] No bidder has incentive to bid in the Auction except doing last-minute `MEV` due to fixed `endBlock` in the [final report](https://code4rena.com/reports/2023-06-stader).

### Proof of Concept

The auction of `SD Token` has a fixed `endBlock`. Bidder(s) would like to get SD Token with the least amount of ETH and they are all incentivized to just bid at the last block, leading to loss of protocol principle during the auction.

```solidity
    function createLot(uint256 _sdAmount) external override whenNotPaused {
        lots[nextLot].startBlock = block.number;
        lots[nextLot].endBlock = block.number + duration;
        lots[nextLot].sdAmount = _sdAmount;

```

Genearally, auctions with a fixed endtime has the known vulnerability of being bid on at the last block. Essentially, the validator/MEVer who has the ability to slip in transaction at the last block has the highest likelihood to get the bid. This basically gives them an advantage and would lead to the auction to end at lower price.

### Recommended Mitigation Steps

Extend the final `endBlock` at each bid. This can be activated at the end of 1h, for example, to ensure the highest bidder can take the auction on in a fair manner.

### Assessed type

MEV

[Link to the original submission](https://github.com/code-423n4/2022-06-stader-findings/issues/70)
