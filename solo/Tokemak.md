# Tokemak, participated in Sherlock (ranked 33rd, 401 USDC)

*********************review commit hash -********************* - **[2f826145cc68409ec8edc9f27ce51dcb43361ce8](https://github.com/sherlock-audit/2023-06-tokemak-judging)**

# Issue H-1: new staker/LMPvault get free earned token on AbstractRewarders

## Summary
new staker get free earned token on AbstractRewarders

## Vulnerability Detail

Here is a PoC, run on test/rewarders/AbstractRewarder.t.sol

```solidity
import { console } from "forge-std/Console.sol";
...
    function test_CalculateEarnedRewardsForGivenWalletWithoutRoll() public {
        _runDefaultScenario();
        vm.roll(block.number + durationInBlock / 2);
        rewarder.stake(address(11111), 10);
        // immediately after staking, earned should be 0
        uint256 earned = rewarder.earned(address(11111));
        console.log("earned reward for a staker immediately after staking : ", earned);
        assertGt(earned, 0);
    }
```

When a 1st time staker call deposit on DestinationVault,
1. `depositUnderlying` through an LMPVault, then `_mint`
2. `_mint` would call `_afterTokenTransfer`, which call `_rewarder.stake(to, amount)`
3. Over on MainRewarder, `_rewarder.stake` would call `_updateReward`
4.  Over on AbstractRewarder, `_updateReward` would assign `rewards[account] = earnRewards`, where earnRewards is `earned(account)`

```solidity
    function earned(address account) public view returns (uint256) {
        return (balanceOf(account) * (rewardPerToken() - userRewardPerTokenPaid[account]) / 1e18) + rewards[account];
    }
```

5. userRewardPerTokenPaid is not initialized as `rewardPerToken()` in the entire workflow for 1st time staker.


## Impact
protocol rewardToken gets drained since:
1. malicious users can use new address/LMPVault (which can be deployed in factory) to conduct staking, once the protocol kicks started and `userRewardPerTokenPaid` grows.
6. honest stakers/re-current stakers quickly lose their share of rewards.
7. rewardToken in the rewarders deposited through notifyReward does not back up the rewards emitted.

=>  loss of protocol core incentive token and staker's APY.

## Code Snippet
https://github.com/sherlock-audit/2023-06-tokemak/blob/main/v2-core-audit-2023-07-14/src/rewarders/AbstractRewarder.sol#L205

## Tool used

Manual Review

## Recommendation
initialized userRewardPerTokenPaid  as `rewardPerToken()` in the workflow for 1st time staker. 

A better/ more consistent solution is to log a new staker in `beforeTokenTransfer`. This issue arises from the difference that Convex marks `earned` for a user before his/her staking, so when a new user has balance of `0`; however in this repo it is calculated post staking the user has positive staked balance already.

Convex

earned is implemented in the same way.
```solidity
    function earned(address account) public view returns (uint256) {
        return
            balanceOf(account)
                .mul(rewardPerToken().sub(userRewardPerTokenPaid[account]))
                .div(1e18)
                .add(rewards[account]);
    }
```

However this is called in updateReward, which is called before executing a user's stake.
```solidity

    function stake(uint256 _amount)
        public
        updateReward(msg.sender)
        returns(bool)
    {
        require(_amount > 0, 'RewardPool : Cannot stake 0');
        
        //also stake to linked rewards
        for(uint i=0; i < extraRewards.length; i++){
            IRewards(extraRewards[i]).stake(msg.sender, _amount);
        }

        _totalSupply = _totalSupply.add(_amount);
        _balances[msg.sender] = _balances[msg.sender].add(_amount);

        stakingToken.safeTransferFrom(msg.sender, address(this), _amount);
        emit Staked(msg.sender, _amount);

        
        return true;
    }

    modifier updateReward(address account) {
        rewardPerTokenStored = rewardPerToken();
        lastUpdateTime = lastTimeRewardApplicable();
        if (account != address(0)) {
            rewards[account] = earned(account);
            userRewardPerTokenPaid[account] = rewardPerTokenStored;
        }
        _;
    }
```

[Link to the original submission](https://github.com/sherlock-audit/2023-06-tokemak-judging/issues/228)


# Issue M-1: the stake flow on gpToke being mandatory on rewarder potentially brick staker's withdrawal from DestinationVault

## Summary
the stake flow on gpToke being mandatory on AbstractRewarder potentially brick staker's withdrawal 

## Vulnerability Detail

Let's look at the withdrawal flow (withdrawUnderlying/withdrawBaseAsset)

1. invoke `_burn(msg.sender, shares)`
2. invoke `_beforeTokenTransfer`, since `_burn` would transfer the token from the user to address(0)
3. invoke `_rewarder.withdraw(from, amount, true);`, where `claim`(the last input) is set to `true`
4. Over to MainRewarder, withdraw would trigger `_processRewards` if claim is set to true, which is indeed the case.
5. `_processRewards` would call `_getReward`
6. Over to AbstractRewarders, `_getReward` would check if the rewardToken is `TOKE`, if so and a non-zero lcokDuration is set, the workflow would attempt to help the user to lock the TOKE reward, calling the function `gptoke.stake`
7. Over to GPToke, stake would call `_stake` which has 3 constraints: 
- A: amount cannot be smaller than `10_000`
- B: amount cannot be larger than `100m * 1e18`
- C: the GPToke cannot be paused.


GPToken::_stake
```solidity
    function _stake(uint256 amount, uint256 duration, address to) internal whenNotPaused {
        //
        // validation checks
        //
        if (to == address(0)) revert ZeroAddress();
        if (amount < MIN_STAKE_AMOUNT) revert StakingAmountInsufficient();
        if (amount > MAX_STAKE_AMOUNT) revert StakingAmountExceeded();

        // duration checked inside previewPoints
        (uint256 points, uint256 end) = previewPoints(amount, duration);

        if (points + totalSupply() > type(uint192).max) {
            revert StakingPointsExceeded();
        }

        // checkpoint rewards for caller
        _collectRewards(to, false);

        // save information for current lockup
        lockups[to].push(Lockup({ amount: uint128(amount), end: uint128(end), points: points }));

        // create points for user
        _mint(to, points);

        emit Stake(to, lockups[to].length - 1, amount, end, points);

        // transfer staked toke in
        toke.safeTransferFrom(msg.sender, address(this), amount);
    }
```

MainRewarder
```solidity
    function withdraw(address account, uint256 amount, bool claim) public onlyStakeTracker {
        _updateReward(account);
        _withdraw(account, amount);

        for (uint256 i = 0; i < extraRewards.length; ++i) {
            IExtraRewarder(extraRewards[i]).withdraw(account, amount);
        }

        if (claim) {
            _processRewards(account, true);
        }
    }
```

DestinationVault
```solidity
    function withdrawUnderlying(uint256 shares, address to) external onlyLMPVault returns (uint256 amount) {
        Errors.verifyNotZero(shares, "shares");
        Errors.verifyNotZero(to, "to");

        amount = shares;

        emit UnderlyingWithdraw(amount, msg.sender, to);

        // Does a balance check, will revert if trying to burn too much
        _burn(msg.sender, shares);

        _ensureLocalUnderlyingBalance(amount);

        IERC20(_underlying).safeTransfer(to, amount);
    }

...

    function _beforeTokenTransfer(address from, address to, uint256 amount) internal virtual override {
        if (from == to) {
            return;
        }

        if (from != address(0)) {
            _rewarder.withdraw(from, amount, true);
        }
    }

```
## Impact
stakers who participate in LMPVault which contributes to DestinationVault where rewardToken is `Toke` can be bricked from withdrawal when GPToke is paused from accepting new stakes, or not enough TOKE is sent in timely as reward to reach the minimum threshold for the staker. 

While the minimum requirement(10_000) is fairly negligible, it may still affect stakers who :
1.) staker who just stake for a very short period of time (1 block).
2.) staker who just have a very small amount of deposit.
3.)  pausing GPToken would pause withdrawal in DestinationVault, thus GMTVault too.

## Code Snippet
https://github.com/sherlock-audit/2023-06-tokemak/blob/main/v2-core-audit-2023-07-14/src/vault/DestinationVault.sol#L341

## Tool used

Manual Review

## Recommendation
There are 2 possible ways to work around this:

1.  change the `claim` input in `_beforeTransfer` to be false, so that user need to claim their only claimable after withdrawal.
2.  make exception on the GPToke::_stake function such that the above constraint does not apply to any TOKE Rewarder.

[Link to the original submission](https://github.com/sherlock-audit/2023-06-tokemak-judging/issues/217)
