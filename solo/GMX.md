# GMX, participated in Sherlock

*********************review commit hash -********************* - **[b1557fa286c35f54c65a38a7b57baf87ecad1b5b](https://github.com/sherlock-audit/2023-02-gmx-judging)**

### The code was reviewed for a total of 6 hours.

# Issue M-1: _validateRange in Config does not check the input value as described

## Summary
_validateRange in Config does not check the input value as described

## Vulnerability Detail
The function `_validateRange` does not check the input against the boundaries.

However in the dev note

```md
    // @dev validate that the value is within the allowed range
    // @param baseKey the base key for the value
    // @param value the value to be set
    function _validateRange(bytes32 baseKey, uint256 value) internal pure {
```

## Impact
Input value that works beyond the expected range could be implemented

## Code Snippet
https://github.com/gmx-io/gmx-synthetics/blob/8028cb8022b85174be861b311f1082b5b76239df/contracts/config/Config.sol#L271

## Recommendation
Check agianst the pre-set maximum and minimum value


[Link to the original submission](https://github.com/sherlock-audit/2023-02-gmx-judging#issue-m-36-_validaterange-in-config-does-not-check-the-input-value-as-described)
