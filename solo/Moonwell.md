# Moonwell, participated in Code4rena

*********************review commit hash -********************* - **[664c57a54571a8af0c765c39009515f8321bc695](https://github.com/code-423n4/2023-07-moonwell-findings)**

# Issue M-1: accrueInterest is expected to revert when the rate is higher than the maximum allowed rate, which is possible since the utilization can be more than 1

Selected as the primary report for [M-15] `accrueInterest` is expected to revert when the rate is higher than the maximum allowed rate, which is possible since the utilization can be more than 1 in the [final report](https://code4rena.com/reports/2023-07-moonwell).

accrueInterest is expected to revert when the rate is higher than the maximum allowed rate, which is possible since the utilization can be more than 1

accrueInterest is an essential function to keep updated of the global mToken interest payment from the borrower to the depositor. The function is called anytime there is a deposit/liquidate/borrow/repay/withdraw.

The function is set to revert when the borrowRateMantissa is bigger than the borrowRateMaxMantissa.

`require(borrowRateMantissa <= borrowRateMaxMantissa, "borrow rate is absurdly high");`.

With proper configuration, this check should be fine since the real borrowRate should be calibrated to be within the maxRate. However, since the utilization could actually be bigger than 1 (when reserve is bigger than cash \[issue-37] , the actual rate could be bigger than the expected unreachable max rate:

Example:<br>
Base: 1%<br>
multiplierPerTimestamp: 5% APR<br>
jumpMultiplier: 50% APR<br>
klink: 50%<br>
We could reasonably assume the max APR is 1% + 5% &ast; 0.5 + 50% &ast; (1-0.5) = 28.5%

However when reserve is more than cash it would cause the utilization be bigger than 1, let'say utilization is 101%: such that the actual rate would  be:

1% + 5% &ast; 0.5 + 50% &ast; (1.01 - 0.5) = 29%

This would cause the accurueInterest to revert.

Impact: all deposit/withdraw/borrow/repay function would fail and severe brick the operation of the protocol and lock user funds. Interest accrual cannot work which impact both borrower for timely exit as well as deposit to collect their interest.

```solidity
    function accrueInterest() virtual override public returns (uint) {
        /* Remember the initial block timestamp */
        uint currentBlockTimestamp = getBlockTimestamp();
        uint accrualBlockTimestampPrior = accrualBlockTimestamp;

        /* Short-circuit accumulating 0 interest */
        if (accrualBlockTimestampPrior == currentBlockTimestamp) {
            return uint(Error.NO_ERROR);
        }

        /* Read the previous values out of storage */
        uint cashPrior = getCashPrior();
        uint borrowsPrior = totalBorrows;
        uint reservesPrior = totalReserves;
        uint borrowIndexPrior = borrowIndex;

        /* Calculate the current borrow interest rate */
        uint borrowRateMantissa = interestRateModel.getBorrowRate(cashPrior, 
        borrowsPrior, reservesPrior);
        require(borrowRateMantissa <= borrowRateMaxMantissa, "borrow rate is 
        absurdly high");
```

<https://github.com/code-423n4/2023-07-moonwell/blob/main/src/core/MToken.sol#L403>

### Utilization can be above 100% when reserve is more than cash

*Note: per [judge request](https://github.com/code-423n4/2023-07-moonwell-findings/issues/40#issuecomment-1676358828), this information from duplicate [issue 37](https://github.com/code-423n4/2023-07-moonwell-findings/issues/37) has been appended to M-16 for the final report.*

Utilization is calculated as `borrow / (cash + borrow - reserve)`. and the utilization would determine the interest rate in this classic jump/klink model.

Consider:
1. both depositor and borrowers come in. Reserves accumulates as interest revenue paid by borrowers accrues.
2. depositor withdraws. The protocol has only borrowers and the reserve.

When reserves becomes more than cash, the formula for utilizationRate would give a utilization rate above 1. This is different from the code comment which said to `@return The utilization rate as a mantissa between [0, 1e18]`

```solidity
     * @return The utilization rate as a mantissa between [0, 1e18]
    function utilizationRate(uint cash, uint borrows, uint reserves) public pure returns (uint) {
        // Utilization rate is 0 when there are no borrows
        if (borrows == 0) {
            return 0;
        }

        return borrows.mul(1e18).div(cash.add(borrows).sub(reserves));
    }
```

https://github.com/code-423n4/2023-07-moonwell/blob/main/src/core/IRModels/JumpRateModel.sol#L66-L75

### Recommended Mitigation Steps

- Consider to cap the real-time borrowRate as the borrowRateMaxMantissa instead of reverting.
- Consider put a cap of 1e18 to the final output.

[Link to the original submission](https://github.com/code-423n4/2023-07-moonwell-findings/issues/40)


# Issue M-2: a single emissionCap is not suitable for different tokens reward if they have different underlying decimals

Selected as the primary report for [M-16] A single emissionCap is not suitable for different tokens reward if they have different underlying decimals in the [final report](https://code4rena.com/reports/2023-07-moonwell).

At MultiRewardDistributor, different rewards token can be set up for each mToken, which is the assetToken or debtToken of the lending protocol. When a reward token is set up, it's emission speed cannot be bigger than the emissionCap (100 &ast; 1e18 as suggested in the code comment). However since multiple reward tokens may be set up, and they may have different decimal places, for example USDT has a decimal of 6 and GUSD even has a decimal of 2 they are virtually not bounded by such emissionCap.

On the contract, tokens with more than 18 decimal would not be practically emitted. This one-size-fit-all emissionCap might create obstacles for the protocol to effectively manage each rewardTokens.

```solidity
    /// @notice The emission cap dictates an upper limit for reward speed emission speed configs
    /// @dev By default, is set to 100 1e18 token emissions / sec to avoid unbounded
    ///  computation/multiplication overflows
    uint256 public emissionCap;
```

```solidity
    function _updateBorrowSpeed(
        MToken _mToken,
        address _emissionToken,
        uint256 _newBorrowSpeed
    ) external onlyEmissionConfigOwnerOrAdmin(_mToken, _emissionToken) {
        MarketEmissionConfig
            storage emissionConfig = fetchConfigByEmissionToken(
                _mToken,
                _emissionToken
            );

        uint256 currentBorrowSpeed = emissionConfig
            .config
            .borrowEmissionsPerSec;

        require(
            _newBorrowSpeed != currentBorrowSpeed,
            "Can't set new borrow emissions to be equal to current!"
        );
        require(
            _newBorrowSpeed < emissionCap,
            "Cannot set a borrow reward speed higher than the emission cap!"
        );
```

<https://github.com/code-423n4/2023-07-moonwell/blob/main/src/core/MultiRewardDistributor/MultiRewardDistributor.sol#L703-L725>

### Recommended Mitigation Steps

Consider creating emissionCap for each rewardTokens so they can be adapted based on their own decimals.

[Link to the original submission](https://github.com/code-423n4/2023-07-moonwell-findings/issues/19)
