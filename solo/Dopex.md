# Dopex, participated in Code4rena

*********************review commit hash -********************* - **[77e932b3b93ffe510d49586d15c6cb81f20f2bc4](https://github.com/code-423n4/2023-08-dopex-findings)**

# Issue H-1: settle at PerpetualAtlanticVault can be permanently Dos-ed by donation due to the strict balance check requirement at VaultLP::subtractLoss

Judged as a duplicate of [H-03] The settle feature will be broken if attacker arbitrarily transfer collateral tokens to the PerpetualAtlanticVaultLP in the [final report](https://code4rena.com/reports/2023-08-dopex).

[Link to the original submission](https://github.com/code-423n4/2023-08-dopex-findings/issues/161)


# Issue H-2: totalWethDelegated is not updated during withdraw leading to potential underflow in sync function in RpdxV2Core

Judged as a duplicate of [H-08] The peg stability module can be compromised by forcing lowerDepeg to revert in the [final report](https://code4rena.com/reports/2023-08-dopex).

[Link to the original submission](https://github.com/code-423n4/2023-08-dopex-findings/issues/78)


# Issue M-1: purchase of at-the-expiry option is possible creating free option opportunity

Judged as a duplicate of [M-12] User can avoid paying high premium price by correctly timing his bond call in the [final report](https://code4rena.com/reports/2023-08-dopex).

[Link to the original submission](https://github.com/code-423n4/2023-08-dopex-findings/issues/166)
