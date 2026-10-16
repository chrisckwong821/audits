# PoolTogether, participated in Code4rena

*********************review commit hash -********************* - **[ec193297735caa64d472a6d89ef7d77a7cb6196c](https://github.com/code-423n4/2023-07-pooltogether-findings)**

# Issue H-1: liquidate at Vault wrongly uses liquidatable asset to compare with the _amountout which is shareToMint leading to potentially over-mint and under-collateralization

Judged as a duplicate of [H-03] `_amountOut` is representing assets and shares at the same time in the `liquidate` function in the [final report](https://code4rena.com/reports/2023-07-pooltogether).

[Link to the original submission](https://github.com/code-423n4/2023-07-pooltogether-findings/issues/5)
