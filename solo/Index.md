# Index Coop, participated in Sherlock (3rd place, 9,274 USDC)

*********************review commit hash -********************* - **[8876eedc14e727e039a6604d60b9fbaf3a8fcbd2](https://github.com/sherlock-audit/2023-05-Index-judging)**

# Issue H-1: _calculateMaxBorrowCollateral does not check whether the strategy is in Emode or not leading to inaccurate representation of LTV.

## Summary
_calculateMaxBorrowCollateral does not check whether the strategy is in Emode or not

## Vulnerability Detail
While there is integration of aave Emode through the new extension `AaveLeverageStrategyExtension`, the existing function `_calculateMaxBorrowCollateral` does not integrate with this new feature.

eMode would increase the LTV and liquidation threshold of the strategy. if the strategy is set to eMode, with the function `_calculateMaxBorrowCollateral` not incorporating the new configuration, the system would behave as if without the emode being enabled.

```solidity
    function _calculateMaxBorrowCollateral(ActionInfo memory _actionInfo, bool _isLever) internal view returns(uint256) {
        
        // Retrieve collateral factor and liquidation threshold for the collateral asset in precise units (1e16 = 1%)
        ( , uint256 maxLtvRaw, uint256 liquidationThresholdRaw, , , , , , ,) = strategy.aaveProtocolDataProvider.getReserveConfigurationData(address(strategy.collateralAsset));

        // Normalize LTV and liquidation threshold to precise units. LTV is measured in 4 decimals in Aave which is why we must multiply by 1e14
        // for example ETH has an LTV value of 8000 which represents 80%
        if (_isLever) {
            uint256 netBorrowLimit = _actionInfo.collateralValue
                .preciseMul(maxLtvRaw.mul(10 ** 14))
                .preciseMul(PreciseUnitMath.preciseUnit().sub(execution.unutilizedLeveragePercentage));

            return netBorrowLimit
                .sub(_actionInfo.borrowValue)
                .preciseDiv(_actionInfo.collateralPrice);
        } else {
            uint256 netRepayLimit = _actionInfo.collateralValue
                .preciseMul(liquidationThresholdRaw.mul(10 ** 14))
                .preciseMul(PreciseUnitMath.preciseUnit().sub(execution.unutilizedLeveragePercentage));

            return _actionInfo.collateralBalance
                .preciseMul(netRepayLimit.sub(_actionInfo.borrowValue))
                .preciseDiv(netRepayLimit);
        }
    }
```
## Impact
wrong LTV and liquidation threshold might be used.

## Code Snippet
https://github.com/sherlock-audit/2023-05-Index/blob/main/index-coop-smart-contracts/contracts/adapters/AaveLeverageStrategyExtension.sol#L1095-L1119

## Tool used

Manual Review

## Recommendation
add checks on whether the setToken is in eMode by calling `getUserEmode`, if so fetch the eModeConfiguration through`getEmodeCategoryData` with argument from `getUserEmode`. If not in emode(userEmode == 0) , follow the existing path of `getReserveConfigurationData`

[Link to the original submission](https://github.com/sherlock-audit/2023-05-Index-judging/issues/13)


# Issue M-1: no checks on chainlink stale price on _createActionInfo

## Summary
no checks on chainlink stale price on _createActionInfo

## Vulnerability Detail
no checks on chainlink stale price on _createActionInfo

```solidity
    function _createActionInfo() internal view returns(ActionInfo memory) {
        ActionInfo memory rebalanceInfo;

        // Calculate prices from chainlink. Chainlink returns prices with 8 decimal places, but we need 36 - underlyingDecimals decimal places.
        // This is so that when the underlying amount is multiplied by the received price, the collateral valuation is normalized to 36 decimals. 
        // To perform this adjustment, we multiply by 10^(36 - 8 - underlyingDecimals)
        int256 rawCollateralPrice = strategy.collateralPriceOracle.latestAnswer();
        rebalanceInfo.collateralPrice = rawCollateralPrice.toUint256().mul(10 ** strategy.collateralDecimalAdjustment);
        int256 rawBorrowPrice = strategy.borrowPriceOracle.latestAnswer();
        rebalanceInfo.borrowPrice = rawBorrowPrice.toUint256().mul(10 ** strategy.borrowDecimalAdjustment);

        rebalanceInfo.collateralBalance = strategy.targetCollateralAToken.balanceOf(address(strategy.setToken));
        rebalanceInfo.borrowBalance = strategy.targetBorrowDebtToken.balanceOf(address(strategy.setToken));
        rebalanceInfo.collateralValue = rebalanceInfo.collateralPrice.preciseMul(rebalanceInfo.collateralBalance);
        rebalanceInfo.borrowValue = rebalanceInfo.borrowPrice.preciseMul(rebalanceInfo.borrowBalance);
        rebalanceInfo.setTotalSupply = strategy.setToken.totalSupply();

        return rebalanceInfo;
    }

```

## Impact
stale price might be used for calculating index token price

## Code Snippet
https://github.com/sherlock-audit/2023-05-Index/blob/main/index-coop-smart-contracts/contracts/adapters/AaveLeverageStrategyExtension.sol#L889-L907

## Tool used

Manual Review

## Recommendation
add a time threshold check on latestAnswer.

[Link to the original submission](https://github.com/sherlock-audit/2023-05-Index-judging/issues/6)


# Issue M-2: setExecutionSettings would be halt during a rebalance operation that gets stuck due to supply cap is reached at Aave

## Summary
setExecutionSettings would be halt during a rebalance operation that gets stuck due to supply cap is reached at Aave

## Vulnerability Detail
rebalance implement a cap of tradeSize and if the need to rebalance require taking more assets than the maxTradeSize, then twapLeverageRatio would be set to the targeted leverage.
twapLeverageRatio == 0 is required during rebalance.

Consider:

lever is needed during rebalance, the strategy require to borrow more ETH and sell to wstETH
during the 1st call of rebalance the protocol cache the new twapLeverageRatio
However wstETH market in Aave reach supply cap. rebalance/iterateRebalance comes to a halt.
`twapLeverageRatio` remains caching the targeted leverage

`setExecutionSettings` requires a condition in which no rebalance is in progress. With the above case, setExecutionSettings can be halted for an extended period of time until the wstETH market falls under supply cap. 

Worth-noting, at the time of writing this issue, [the wstETH market at Aave has been at supply cap](https://app.aave.com/reserve-overview/?underlyingAsset=0x7f39c581f595b53c5cb19bd0b3f8da6c935e2ca0&marketName=proto_mainnet_v3)

In this case, malicious actor who already has a position in wstETH can do the following:
- deposit into the setToken, trigger a rebalance.

- malicious trader withdraw his/her position in wstETH so there opens up vacancy for supply again.

- protocol owner see supply vacancy, call rebalance in order to lever as required. Now `twapLeverageRatio` is set to new value since multiple trades are needed

- malicious trade now re-supply the wstETH market at Aave so it reaches suppply cap.

- the protocol gets stuck with a non-zero twapLeverageRatio, setExecutionSettings can not be called.


```solidity
    function setExecutionSettings(ExecutionSettings memory _newExecutionSettings) external onlyOperator noRebalanceInProgress {
        execution = _newExecutionSettings;

        _validateNonExchangeSettings(methodology, execution, incentive);

        emit ExecutionSettingsUpdated(
            execution.unutilizedLeveragePercentage,
            execution.twapCooldownPeriod,
            execution.slippageTolerance
        );
    }
```
## Impact
setExecution would be halt during a rebalance operation that gets stuck due to supply cap is reached at Aave

## Code Snippet
https://github.com/sherlock-audit/2023-05-Index/blob/main/index-coop-smart-contracts/contracts/adapters/AaveLeverageStrategyExtension.sol#L466-L476
## Tool used

Manual Review

## Recommendation

Add some checks on whether the supply cap of an Aave market is reached during a rebalance. If so, allows a re-set of twapLeverageRatio

[Link to the original submission](https://github.com/sherlock-audit/2023-05-Index-judging/issues/9)
