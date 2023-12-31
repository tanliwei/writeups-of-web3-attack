### Summary
On Nov 1, 2023, our MetaScout [detected](https://x.com/MetaTrustAlert/status/1719660132240691257?s=20) that the lending protocol, Onyx Protocol on Ethereum, was under a flash loan attack with a loss of $2.1M. The root causes are the proposal of adding a new market that was first executed by the hacker, and the precision loss issue of the compound-fork protocol. 
MetaTrust Labs conducted in-depth research and analysis on the exploit, revealing how hackers used governance proposals and protocol vulnerability to attack Onyx Protocol.

### Onyx Protocol
Onyx Protocol(https://docs.onyx.org/) is an algorithmic money market designed to bring secure and trustless credit and lending to users on the Ethereum Network. 
![image.png](https://storage.googleapis.com/metatrust/website/post/image/1698907360943_image.png)
On Oct 29, 2023, Onyx protocol(https://x.com/OnyxProtocol/status/1718348637158137858?s=20) launched a proposal OIP-22 to add $PEPE to the market. However, it is unfortunately targeted and executed by the hacker for the exploiting.
![image.png](https://storage.googleapis.com/metatrust/website/post/image/1698907434080_image.png)
The Onyx protocol turned out to be a compound-forked protocol after we checked its on-chain deployed contracts, and its TVL dropped from $2.86M to $557K due to yesterday's attack.
![image.png](https://storage.googleapis.com/metatrust/website/post/image/1698907444856_image.png)

### Attacking Timeline
| Time | Action | Transaction |
| --- | --- | --- |
| 2023-10-28 02:29:11 PM +UTC | The proposal OIP-22 was created to add a new market  | 0x309f437ece123ee77b5cc2435783578424bb4e30448dedf6f845bd0c2a4e40a6 |
| 2023-11-01 **09:57:47**  AM +UTC | The hacker executed proposal OIP-22 | 0xb91e0731a0f1eb0abf884717a065f7282a5daaacd172a0db8d4a61609c978ed8 |
| 2023-11-01 **09:58:47**  AM +UTC | Launched the attack only 1 minute later than the execution of proposal 22. Note that the hacker is the one who first interacts with the new market.  | 0xf7c21600452939a81b599017ee24ee0dfd92aaaccd0a55d02819a7658a6ef635 0x27a3788d504af542681436bfdecf1823f7a8a691d04309ad33e6d3825e899746  |

### Attack Loss 
The total loss of two attacking transactions is ~$2.14M.
| Transaction | Amount |
| --- | --- |
| 0xf7c21600452939a81b599017ee24ee0dfd92aaaccd0a55d02819a7658a6ef635 | 1,156.9 $ETH(worth $2.088M) |
| 0x27a3788d504af542681436bfdecf1823f7a8a691d04309ad33e6d3825e899746 | $60K worth assets, including $MATIC, $UNI, $APE, $USDP, and $WETH. |

### Attackers
- 0x085bDfF2C522e8637D4154039Db8746bb8642BfF
- 0x5083956303a145f70ba9f3d80c5e6cb5ac842706
### Attacking Contract
0x052ad2f779c1b557d9637227036ccaad623fceaa
### Attacked Contract
- Proxy contract: https://etherscan.io/address/0x5fdbcd61bc9bd4b6d3fd1f49a5d253165ea11750
- Implementation Contract: https://etherscan.io/address/0x9dcb6bc351ab416f35aeab1351776e2ad295abc4#code
### Governance Contract
https://etherscan.io/address/0xdec2f31c3984f3440540dc78ef21b1369d4ef767
### Attacking Steps
TL;DR

Take one of the attacking transactions, 0xf7c216, as an example.
1. The hacker(0x085bDf) executes proposal OIP-22 first to add a new market oPEPE(0x5fdbcd).
![image.png](https://storage.googleapis.com/metatrust/website/post/image/1698907522497_image.png)
![image.png](https://storage.googleapis.com/metatrust/website/post/image/1698907532668_image.png)
2. One minute after the new market is added, launches a flash loan from AAVE and gets 4,000 $WETH
    1. Swaps 4,000 $WETH to 2,520,870,348,093 $PEPE
    2. Transfers all the $PEPE to the address 0xf8e153
![image.png](https://storage.googleapis.com/metatrust/website/post/image/1698907856990_image.png)
    3. Creates contract at the above address 0xf8e153, mints 50,000,000,000,000,000,000 $oPEPE with 1 $PEPE, redeems most of $oPEPE and leaves only **2 wei** to the market oPEPE.
    4. Transfers 2,520,870,348,093 $PEPE to the market oPEPE and enters the market with $oPEPE
    5. Borrow 334 $ETH
    6. redeem only 1 wei $oPEPE for 2,520,870,348,093 $PEPE due to precision loss issue.
        1. exchangeRate = (totalCash + totalBorrows - totalReserves) / totalSupply 
= 2,520,870,348,093,423,681,390,050,791,472 / 2 
= 1,260,435,174,046,711,840,695,025,395,**736**
        2. redeemAmountIn = 2,520,870,348,093,423,681,390,050,791,**470**
        3. redeemTokens = redeemAmountIn / exchangeRate = 1, due to the truncation. 
![image.png](https://storage.googleapis.com/metatrust/website/post/image/1698907716744_image.png)
        4. Liquidates borrower(0xf8e153)'s 881,647,840 wei $PEPE
        5. Redeems 856,961,701 wei $PEPE
  7. Repeats above steps from step b. to step f. borrows $USDC, $USDT, $PAXG, $DAI, $WBTC, and $LINK, and swaps them for $ETH.
3. Repays the AAVE flash loan with 4,002 $WETH and get a profit of 1156.9 $ETH.

### Root Cause
- On the one hand, the hacker is very familiar with the precision loss issue of compound-fork protocols and noticed the bug of Onyx protocol in advance, thus, the hacker probably monitors proposal OIP-22, once the proposal is active and ready to be executed, the hacker firstly executes it, only one minute later, launches the attack.
- On the other hand, the precision loss bug is the root cause of the attack. The hacker manipulates the `totalSupply` to be a tiny value, 2, and increases the totalCash to be extremely high value, 2520870348093423681390050791471, to amplify the `exchangeRate`, which results in calculation truncation when redeeming.
```solidity
    function exchangeRateStoredInternal() internal view returns (MathError, uint) {
        //...
        /*
        *  exchangeRate = (totalCash + totalBorrows - totalReserves) / totalSupply
        */
        uint totalCash = getCashPrior();
        uint cashPlusBorrowsMinusReserves;
        Exp memory exchangeRate;
        MathError mathErr;

        (mathErr, cashPlusBorrowsMinusReserves) = addThenSubUInt(totalCash, totalBorrows, totalReserves);
        if (mathErr != MathError.NO_ERROR) {
            return (mathErr, 0);
        }

        (mathErr, exchangeRate) = getExp(cashPlusBorrowsMinusReserves, _totalSupply);
        if (mathErr != MathError.NO_ERROR) {
            return (mathErr, 0);
        }
        //...
    }
```
### Recommendation
1. Do a detailed audit of the governance proposal, not limited to the smart contract, especially for the initialization scenario and other edge cases.
2. Consider adding a small amount of shares into the market initialization to prevent manipulation, especially for the compound-fork protocols.
3. Recommend adopting the monitoring system and pausing the protocol when emergency cases happen, if there is a monitoring system for Onyx, the second attack transaction that happened more than half an hour later could be prevented to decrease the loss. Alternatively, integrating a mempool blocking system would be beneficial. This system can effectively detect attack transactions in the mempool when attackers are executing their attacks, allowing for preemptive blocking to avoid losses.
 
### Fund Flow
As of the time of writing, attack(0x085bDf) transfers 1140 $ETH to the money-laundry protocol Tornado.Cash with another controlled address(0x4c9c86).
![image.png](https://storage.googleapis.com/metatrust/website/post/image/1698907665670_image.png)
Another attacker still keeps stolen tokens in the same wallet(0x508395).

### Similar Attack
The bug is similar to [the previous hack](https://x.com/HundredFinance/status/1647247792589471745?s=20) of Hundred Finance on 2023.04.15, which resulted in a loss of approximately $7 million.

TX: 0x6e9ebcdebbabda04fa9f2e3bc21ea8b2e4fb4bf4f4670cb8483e2f0b2604f451

### About MetaTrust Labs
MetaTrust Labs is a leading provider of Web3 AI security tools and code auditing services incubated at Nanyang Technological University, Singapore. We provide advanced AI solutions that empower developers and project stakeholders to protect Web3 applications and smart contracts. At MetaTrust Labs, we are committed to protecting the Web3 space so that builders can innovate with confidence and reliability.

Website: https://metatrust.io/company/blogs/post/how-onyxs-governance-and-vulnerability-became-hackers-golden-shovels

Twitter: https://x.com/DanielSlothx/status/1720033194781913180?s=20 