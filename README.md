#  Security audits

Competitive smart-contract security reviews as **0xGoodess** on [Sherlock](https://audits.sherlock.xyz/watson/0xGoodess) and **0xWaitress** on [Code4rena](https://code4rena.com/@0xWaitress).

- 13 High and 39 Medium valid findings across 30+ protocols, USD 41K+ awarded (platform profile totals)
- 3rd place on Index Coop and 4th on Perennial (Sherlock)
- Findings in EigenLayer, GMX, Notional, Moonwell, PoolTogether and Maia

Each file below holds my findings for one contest, with a link to the original submission. This page lists 30 contests (11 High, 35 Medium) from published reports.

## Highlights

### 1. Index Coop: leverage engine ignored Aave's eMode risk settings
**High · Sherlock, 3rd place overall · 9,274 USDC awarded for the contest**

Index Coop's leveraged ETH strategy borrows on Aave, and it calculates how much it can safely borrow or repay from Aave's loan-to-value limits. The new extension turned on Aave's eMode, which raises those limits. The old calculation still read the standard limits, so the strategy worked from the wrong risk numbers. I recommended checking whether the position is in eMode and, if so, using eMode's settings.

[Full write-up](https://github.com/chrisckwong821/audits/blob/main/solo/Index.md) · [Original submission](https://github.com/sherlock-audit/2023-05-Index-judging/issues/13)

### 2. Perennial: anyone could delay another user's deposit indefinitely
**High · Sherlock, 4th place overall · 6,608 USDC awarded for the contest**

Perennial's vault settles deposits in the next epoch to block front-running. But anyone could deposit on someone else's behalf, and each new deposit pushed that account's settlement back an epoch. An attacker could keep sending dust deposits to delay a victim's funds until settlement favored the attacker. I recommended allowing deposits only from the account owner or an address it approves.

[Full write-up](https://github.com/chrisckwong821/audits/blob/main/solo/Perennial.md) · [Original submission](https://github.com/sherlock-audit/2023-05-perennial-judging/issues/33)

### 3. Moonwell: a safety cap that could freeze the whole lending market
**Medium · Code4rena, selected as the primary report**

Moonwell reverts any interest update if the borrow rate exceeds a maximum, as a safety check. I showed that utilization can go above 100% when reserves exceed available cash, which pushes the rate past the "unreachable" maximum. Because interest updates run on every deposit, withdrawal, borrow and repayment, all of them would fail and user funds would be locked. The judges selected my report as the primary one for this issue.

[Full write-up](https://github.com/chrisckwong821/audits/blob/main/solo/Moonwell.md) · [Final report](https://code4rena.com/reports/2023-07-moonwell) · [Original submission](https://github.com/code-423n4/2023-07-moonwell-findings/issues/40)

## Solo

### March 2023

- [Telcoin - defi for mobile phone](https://github.com/chrisckwong821/audits/blob/main/solo/telecoin.md) — Sherlock, ranked 5th · 1 Medium
- [GMX - 2.0](https://github.com/chrisckwong821/audits/blob/main/solo/GMX.md) — Sherlock, ranked 18th · 1 Medium
- [Notional V3](https://github.com/chrisckwong821/audits/blob/main/solo/Notional-V3.md) — Sherlock, ranked 9th · 1 Medium
- [Asymmetry Finance](https://github.com/chrisckwong821/audits/blob/main/solo/Asymmetry.md) — Code4rena · 1 Medium

### April 2023

- [Frankencoin](https://github.com/chrisckwong821/audits/blob/main/solo/Frankencoin.md) — Code4rena, ranked 6th · 2 Medium
- [GMX - update](https://github.com/chrisckwong821/audits/blob/main/solo/GMX-0423.md) — Sherlock, ranked 8th · 1 Medium
- [Teller](https://github.com/chrisckwong821/audits/blob/main/solo/Teller.md) — Sherlock, ranked 46th · 2 Medium
- [EigenLayer](https://github.com/chrisckwong821/audits/blob/main/solo/EigenLayer.md) — Code4rena, ranked 18th · 1 Medium
- [Rubicon v2](https://github.com/chrisckwong821/audits/blob/main/solo/Rubicon.md) — Code4rena, ranked 71st · 1 Medium

### May 2023

- [Ajna](https://github.com/chrisckwong821/audits/blob/main/solo/Ajna-0523.md) — Code4rena · 1 High
- [Perennial](https://github.com/chrisckwong821/audits/blob/main/solo/Perennial.md) — Sherlock, ranked 4th · 1 High, 1 Medium
- [Index Coop](https://github.com/chrisckwong821/audits/blob/main/solo/Index.md) — Sherlock, 3rd place · 1 High, 2 Medium
- [Footium](https://github.com/chrisckwong821/audits/blob/main/solo/Footium.md) — Sherlock, ranked 27th · 2 Medium

### June 2023

- [Symmetrical](https://github.com/chrisckwong821/audits/blob/main/solo/Symmetrical.md) — Sherlock, ranked 35th · 1 Medium
- [Unitas Protocol](https://github.com/chrisckwong821/audits/blob/main/solo/Unitas.md) — Sherlock, ranked 16th · 2 Medium
- [Stader Labs](https://github.com/chrisckwong821/audits/blob/main/solo/Stader.md) — Code4rena · 2 Medium

### July 2023

- [Tokemak](https://github.com/chrisckwong821/audits/blob/main/solo/Tokemak.md) — Sherlock, ranked 33rd · 1 High, 1 Medium
- [Amphora Protocol](https://github.com/chrisckwong821/audits/blob/main/solo/Amphora.md) — Code4rena · 1 High
- [Arcade.xyz](https://github.com/chrisckwong821/audits/blob/main/solo/Arcade.md) — Code4rena · 1 Medium
- [Moonwell](https://github.com/chrisckwong821/audits/blob/main/solo/Moonwell.md) — Code4rena · 2 Medium
- [PoolTogether](https://github.com/chrisckwong821/audits/blob/main/solo/PoolTogether.md) — Code4rena · 1 High
- [Tapioca DAO](https://github.com/chrisckwong821/audits/blob/main/solo/Tapioca.md) — Code4rena · 3 Medium

### August 2023

- [Dopex](https://github.com/chrisckwong821/audits/blob/main/solo/Dopex.md) — Code4rena · 2 High, 1 Medium

### September 2023

- [Maia DAO (Ulysses)](https://github.com/chrisckwong821/audits/blob/main/solo/Maia.md) — Code4rena · 1 High

### October 2023

- [LooksRare](https://github.com/chrisckwong821/audits/blob/main/solo/LooksRare.md) — Sherlock, ranked 8th · 1 Medium
- [Canto](https://github.com/chrisckwong821/audits/blob/main/solo/Canto.md) — Code4rena · 1 High
- [Ethena Labs](https://github.com/chrisckwong821/audits/blob/main/solo/Ethena.md) — Code4rena · 1 Medium
- [NextGen](https://github.com/chrisckwong821/audits/blob/main/solo/NextGen.md) — Code4rena · 1 High
- [Open Dollar](https://github.com/chrisckwong821/audits/blob/main/solo/OpenDollar.md) — Code4rena · 1 Medium

### January 2024

- [Salty.IO](https://github.com/chrisckwong821/audits/blob/main/solo/Salty.md) — Code4rena · 3 Medium
