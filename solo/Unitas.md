# Unitas Protocol, participated in Sherlock (ranked 16th, 169 USDC)

*********************review commit hash -********************* - **[0212cb9a37359335bb5786830c56b622072c00b1](https://github.com/sherlock-audit/2023-04-unitasprotocol-judging)**

# Issue M-1: user has no way to express minimum output acceptance during swap in Unitas

## Summary
user has no way to express minimum output acceptance during swap in Unitas.

## Vulnerability Detail
user has no way to express minimum output acceptance during swap in Unitas. Since there is also no expiry control on the function call, if the price conversion of the inToken and outToken changes, it would also lead to potential unexpected fund loss to users. 

```solidity
function swap(address tokenIn, address tokenOut, AmountType amountType, uint256 amount)
        external
        whenNotPaused
        nonReentrant
        returns (uint256 amountIn, uint256 amountOut)
    {
        IERC20Token feeToken;
        uint256 fee;
        uint24 feeNumerator;
        uint256 price;
        ITokenManager.PairConfig memory pair = tokenManager.getPair(tokenIn, tokenOut);

        (amountIn, amountOut, feeToken, fee, feeNumerator, price) = _getSwapResult(pair, tokenIn, tokenOut, amountType, amount);

        _require(IERC20(tokenIn).balanceOf(msg.sender) >= amountIn, Errors.BALANCE_INSUFFICIENT);

        _swapIn(tokenIn, msg.sender, amountIn);

        _swapOut(tokenOut, msg.sender, amountOut);

        if (fee > 0) {
            address feeReceiver = surplusPool;
            feeToken.mint(feeReceiver, fee);
            emit SwapFeeSent(address(feeToken), feeReceiver, fee);
        }

        _checkReserveRatio(tokenOut == pair.baseToken ? pair.buyReserveRatioThreshold : pair.sellReserveRatioThreshold);

        emit Swapped(tokenIn, tokenOut, msg.sender, amountIn, amountOut, address(feeToken), fee, feeNumerator, price);
    }
```

## Impact
no minimum output protection leads to potential unexpected fund loss to users during swap. 

## Code Snippet
https://github.com/sherlock-audit/2023-04-unitasprotocol/blob/main/Unitas-Protocol/src/Unitas.sol#L208-L237

## Tool used

Manual Review

## Recommendation
adding a minimumOutput argument for user to express the acceptable output for the swap

[Link to the original submission](https://github.com/sherlock-audit/2023-04-unitasprotocol-judging/issues/34)


# Issue M-2: There is no liveness check on the getLatestPrice from oracle when used on Unitas

## Summary
There is no liveness check on the getLatestPrice from oracle when used on Unitas

## Vulnerability Detail
XOracle provides price feed of assets for Unitas to use during swap. the amount in/out would be calculated based on the latestPrice from XOracle. However, there is no liveness check on the return value from the latestPrice. If the price feed is outdated or the chain just return online after a long time offline, the swap would be done with the outdated price

```solidity
    function _getSwapResult(
        ITokenManager.PairConfig memory pair,
        address tokenIn,
        address tokenOut,
        AmountType amountType,
        uint256 amount
    )
        internal
        view
        returns (uint256 amountIn, uint256 amountOut, IERC20Token feeToken, uint256 fee, uint24 feeNumerator, uint256 price)
    {
        _checkAmountPositive(amount);

        // Checks the tokens of the pair config are valid
        bool isBuy = tokenOut == pair.baseToken;
        _require(
            (isBuy && tokenIn == pair.quoteToken) ||
                (tokenOut == pair.quoteToken && tokenIn == pair.baseToken),
            Errors.PAIR_INVALID
        );

        address priceQuoteToken = _getPriceQuoteToken(tokenIn, tokenOut);
        price = oracle.getLatestPrice(priceQuoteToken);
```
## Impact
outdated price feed would still be used in Unitas during swap

## Code Snippet
https://github.com/sherlock-audit/2023-04-unitasprotocol/blob/main/Unitas-Protocol/src/Unitas.sol#L417-L440

## Tool used

Manual Review

## Recommendation
add a liveness check to make sure the price is within certain threshold.

[Link to the original submission](https://github.com/sherlock-audit/2023-04-unitasprotocol-judging/issues/54)
