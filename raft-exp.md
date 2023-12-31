### Summary
2023-11-11 02:59:23 a.m. UTC +8:00, MetaScout detected that the stablecoin protocol on #Ethereum, Raft, was under a flash loan attack. It resulted in ~6.7m stablecoin $R being minted and the protocol lost $3.6M. The root cause is the precision calculation issue when minting share tokens, which is used by the hacker to get extra share tokens.

MetaTrust Labs conducted in-depth research and analysis on the exploit, revealing how the hacker exploits vulnerability.
### The Stablecoin Protocol Raft
Raft is a DeFi protocol that lets you generate R by depositing liquid staking tokens (LSDs) as collateral, providing a capital-efficient way to borrow while keeping your staking rewards. https://raft.fi/.
![image.png](https://storage.googleapis.com/metatrust/website/post/image/1699700681338_image.png)
As of the time of writing, its TVL has dropped 46% to $7M after today's attack. The price of $R has dropped 99.6% to $0.0036
![image.png](https://storage.googleapis.com/metatrust/website/post/image/1699701052176_image.png)


### Transaction
https://etherscan.io/tx/0xfeedbf51b4e2338e38171f6e19501327294ab1907ab44cfd2d7e7336c975ace7
### Attacker
0xc1f2b71a502b551a65eee9c96318afdd5fd439fa
### Attacking Contract
0x0a3340129816a86b62b7eafd61427f743c315ef8
### Vulnerable Contract
InterestRatePositionManager: 0x9ab6b21cdf116f611110b048987e58894786c244

### Attacking Steps
1. Lend 6000 $cbETH from AAVE with the flash loan ;
2. Transfer a total of 6001 $cbETH to the InterestRatePositionManager contract;
3. Liquidate a pre-created position 0x011992114806e2c3770df73fa0d19884215db85f on the InterestRatePositionManager contract:
![image.png](https://storage.googleapis.com/metatrust/website/post/image/1699701018269_image.png)
4. Set the index of the raft collateral indexable token to 6,003,441,032,036,096,684,181, which is the $cbETH balance of the InterestRatePositionManager contract and amplified more than 1000 times due to the donation of step 2:
![image.png](https://storage.googleapis.com/metatrust/website/post/image/1699701003445_image.png)
5. 5. Mint 1 wei share with only 1 wei $cbETH, due to the divUp function being used when calculating the share. Note that, the minimal value returned from the divUp function is 1 when the numerator is non-zero no matter how big the denominator is:
![image.png](https://storage.googleapis.com/metatrust/website/post/image/1699701009191_image.png)
6. Repeat step 5 60 times to get 60 wei shares, which is 10,050 $cbETH;
7. Redeem 6003 $cbETH with only 90 wei $rcbETH-c;
![image.png](https://storage.googleapis.com/metatrust/website/post/image/1699700994927_image.png)
8. Borrow 6.7M $R, which is profit, and finally swapped for 1575 $ETH(worth $3.6M) with different Dapps, including: 
  - Swap 2.1M $R for 2M $sDAI on Balancer;
  - Swap 1.2M $R for 1.15 $DAI on Balancer;
  - Swap 200K $R for 86K $USDC on Uniswap.
![image.png](https://storage.googleapis.com/metatrust/website/post/image/1699700988253_image.png)
9. What is unbelievable is that the hacker burned 1570 $ETH to the blackhole address, which means no profit 
![image.png](https://storage.googleapis.com/metatrust/website/post/image/1699700982172_image.png)
### Root Cause
The root cause is the precision calculation issue when minting share tokens, which is used by the hacker to get extra share tokens. Because the index was amplified by the donation of $cbETH, the hacker's shares are worth more value, so the hacker can redeem a tiny $rcbETH-c for 6003  $cbETH and borrow tons of $R.
### Key Code
```solidity
    function divUp(uint256 a, uint256 b) internal pure returns (uint256) {
        if (a == 0) {
            return 0;
        } else {
            return (((a * ONE) - 1) / b) + 1;
        }
    }
    
    function mint(address to, uint256 amount) public virtual override onlyPositionManager {
        _mint(to, amount.divUp(storedIndex));
    }
```
### Asset Loss
$3.4M
### Fund flow 
1570 $ETH was burned by the mistake of the hacker.
![image.png](https://storage.googleapis.com/metatrust/website/post/image/1699700961999_image.png)
As of the time of writing, there are 1.4M $R tokens (worth $4.6K) staying in the attacker's wallet.
![image.png](https://storage.googleapis.com/metatrust/website/post/image/1699700968212_image.png)
### Recommandation
1. Consider checking the potential rounding issue in the case of rate calculation, if it is manipulatable by the malicious user in edge cases, like the Raft exploit case.
2. Recommend adopting the monitoring system and pausing the protocol when emergency cases happen. Alternatively, integrating a mempool blocking system would be beneficial. This system can effectively detect attack transactions in the mempool when attackers are executing their attacks, allowing for preemptive blocking to avoid losses.

### About MetaTrust Labs
MetaTrust Labs is a leading provider of Web3 AI security tools and code auditing services incubated at Nanyang Technological University, Singapore. We provide advanced AI solutions that empower developers and project stakeholders to protect Web3 applications and smart contracts. At MetaTrust Labs, we are committed to protecting the Web3 space so that builders can innovate with confidence and reliability.

Website: https://metatrust.io/company/blogs/post/when-hacking-goes-haywire-rafts-1570-eth-loss-takes-a-cosmic-detour-to-the-black-hole

Twitter: https://x.com/DanielSlothx/status/1723334206787592378?s=20
