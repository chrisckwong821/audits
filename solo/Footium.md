# Footium, participated in Sherlock (ranked 27th, 73 USDC)

*********************review commit hash -********************* - **[9ed563e3dd1502cdad0b5af568d78614daa6c082](https://github.com/sherlock-audit/2023-04-footium-judging)**

# Issue M-1: FootiumEscrow uses transfer which may fail for non standard ERC20

## Summary
FootiumEscrow uses transfer which may fail for non standard ERC20

## Vulnerability Detail
```solidity
    function transferERC20(
        IERC20 erc20Contract,
        address to,
        uint256 amount
    ) external onlyClubOwner {
        erc20Contract.transfer(to, amount); @> audit
    }
```
## Impact
transfer fails for non-standard ERC20

## Code Snippet
https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumEscrow.sol#L105-L111
## Tool used

Manual Review

## Recommendation
```solidity
    function transferERC20(
        IERC20 erc20Contract,
        address to,
        uint256 amount
    ) external onlyClubOwner {
        erc20Contract.safeTransfer(to, amount); @> audit
    }
```

[Link to the original submission](https://github.com/sherlock-audit/2023-04-footium-judging/issues/257)


# Issue M-2: FootiumEscrow uses approve which may fail for non standard ERC20

## Summary
FootiumEscrow uses approve which may fail for non standard ERC20

## Vulnerability Detail
```solidity
    function setApprovalForERC20(
        IERC20 erc20Contract,
        address to,
        uint256 amount
    ) external onlyClubOwner {
        erc20Contract.approve(to, amount); @>audit
    }
```
## Impact
FootiumEscrow uses approve which may fail for non standard ERC20

## Code Snippet
https://github.com/sherlock-audit/2023-04-footium/blob/main/footium-eth-shareable/contracts/FootiumEscrow.sol#L75-L81

## Tool used

Manual Review

## Recommendation
```solidity
    function setApprovalForERC20(
        IERC20 erc20Contract,
        address to,
        uint256 amount
    ) external onlyClubOwner {
        erc20Contract.safeApprove(to, amount); @>audit
    }
```

[Link to the original submission](https://github.com/sherlock-audit/2023-04-footium-judging/issues/258)
