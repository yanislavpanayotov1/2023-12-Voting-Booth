### [H-1] Rewards miscalculation in `_distributeRewards` causing voters `for` to get less rewards

**Description:** In `VotingBooth.sol::_distributeRewards` `rewardsPerVoter` is miscalculated and is causing two things:

1. Voters to receive less rewards than intended
2. Causing Eth to get stuck in the contract, because it is counting the `against` voters as well `for` voters

```javascript
    function _distributeRewards() private {
            uint256 totalVotesFor = s_votersFor.length;
            uint256 totalVotesAgainst = s_votersAgainst.length;
            uint256 totalVotes = totalVotesFor + totalVotesAgainst;

        
            uint256 totalRewards = address(this).balance;

            
            if (totalVotesAgainst >= totalVotesFor) {
               
                _sendEth(s_creator, totalRewards);
            }
            
            else {
@>              uint256 rewardPerVoter = totalRewards / totalVotes;

                for (uint256 i; i < totalVotesFor; ++i) {
                    
                    if (i == totalVotesFor - 1) {
                        rewardPerVoter = Math.mulDiv(totalRewards, 1, totalVotes, Math.Rounding.Ceil);
                    }
                    _sendEth(s_votersFor[i], rewardPerVoter);
                }
            }
        }
```

**Impact:** This miscalculation causes the voters to receive significantly less rewards, as well as causing eth to get stuck in the contract indefinitely.

**Proof of Concept:**

<details>
<summary>PoC</summary>

```javascript
    function testUsersWillReceiveLessEthAndExtraFundStuckInContractForever() public {
        console2.log("Voting booth balance before voting: ", address(booth).balance);
        console2.log("User 1 balance before voting: ", address(0x1).balance);
        vm.startPrank(address(0x1));
        booth.vote(true);
        console2.log("User 2 balance before voting: ", address(0x2).balance);
        vm.startPrank(address(0x2));
        booth.vote(true);

        vm.startPrank(address(0x3));
        booth.vote(false);

        console2.log("User 1 balance after voting: ", address(0x1).balance);
        console2.log("User 2 balance after voting: ", address(0x2).balance);
        console2.log("Voting booth balance after voting: ", address(booth).balance);
        
    }
```    
</details>

**Recommended Mitigation:** We recommend changing `rewardsPerVoter` to use only the `for` voters

```diff
-        uint256 rewardPerVoter = totalRewards / totalVotes;
+        uint256 rewardPerVoter = totalRewards / totalVotesFor;
```