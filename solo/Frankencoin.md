# Frankecoin, participated in Code4Arena (ranked 6th overall with 2k pay)

*********************review commit hash -********************* - **[1022cb106919fba963a89205d3b90bf62543f68f](https://github.com/code-423n4/2023-04-frankencoin)**

### The code was reviewed for a total of 9 hours.

# Issue M-1(Solo): notifyLoss can be frontrun by redeem

## Summary
notifyLoss can be frontrun by redeem

## Vulnerability Detail

notifyLoss immediately transfer zchf from reserve to the minter, reducing the amount of reserve and hence the equity and zchf to claim pershare.

While the deposit has 90 days cooldown before depositor can withdraw, current depositor that passed this cooldown can take advantage of a notifyLoss event by first frontrunning the notifyLoss by redeeming, then re-depositing into the protocol to take advantage of the reducedvalue per share.

notifyLoss can only be called by MintingHub::end, current depositor can bundle `redeem + end + deposit`, when they see a challenge that is ending in loss for the reserve.

## Impact
notifyLoss can be frontrun by redeem

## Code Snippet
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Frankencoin.sol#L280-L288

## Recommendation
This is a re-current issue for most defi strategy to account loss in a mev-resistent way, a few possible solutions:

1. create an additional window for MintingHub::end to be called by a whitelist, before it opens up to the public. The whitelist is trusted bot that will call `end` through private mempool.
2. amortised the loss in the next coming period of time instead in 1 go with a MAX_SPEED.
3. create an withdrawal queue such that the final withdrawal price is dependent on the upcoming equity change(s)


# Issue M-2: restructureCapTable does not enumerate each addressToWipe

## Summary
restructureCapTable does not enumerate each addressToWipe

# Vulnerability details
restructureCapTable does not enumerate each addressToWipe

## Impact
restructureCapTable does not enumerate each addressToWipe

## Proof of Concept
https://github.com/code-423n4/2023-04-frankencoin/blob/main/contracts/Equity.sol#L313

```solidity
function restructureCapTable(address[] calldata helpers, address[] calldata addressesToWipe) public {
    require(zchf.equity() < MINIMUM_EQUITY);
    checkQualified(msg.sender, helpers);
    for (uint256 i = 0; i<addressesToWipe.length; i++){
        address current = addressesToWipe[0]; @> audit
        _burn(current, balanceOf(current));
    }
}
```
## Tools Used

## Recommended Mitigation Steps

```solidity
function restructureCapTable(address[] calldata helpers, address[] calldata addressesToWipe) public {
    require(zchf.equity() < MINIMUM_EQUITY);
    checkQualified(msg.sender, helpers);
    for (uint256 i = 0; i<addressesToWipe.length; i++){
        address current = addressesToWipe[i];
        _burn(current, balanceOf(current));
    }
}
```
