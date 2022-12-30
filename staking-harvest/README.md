# Staking / Harvest

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

**Staking is currently available only on Liquidswap Testnet. We will make a separate announcement once it is released on the mainnet.** :warning:****

**Why launch Harvest?**

The objective of Liquidswap’s Harvest staking program is to incentivize liquidity providers by rewarding them with APT and third-party coins (tokens) issued on Aptos.&#x20;

Staking is easy to use, fully permissionless, and independent at the same time. Other projects can use the Harvest solution to configure and launch their own staking pools. You don’t even have to use Liquidswap LPs: any assets issued on Aptos can be used for staking and as rewards.

The staking contracts are written in Move. The key features are:

* The contracts are permissionless: anyone can create a new staking pool.
* Any coins\* can be used as the staking or reward asset.
* NFTs can be used as an additional parameter for the pool to provide reward boosts to the users holding specified NFTs.
* Rewards can be deposited into the pool at any time to extend the harvesting period.
* Users can unstake their coins without losing any of the profit after 1 week of staking - or once the harvesting period is complete.
* Users can harvest (withdraw) the accumulated rewards at any time.

The easiest way to use the staking protocol is via the[ Liquidswap Dapp](https://liquidswap.com/). Alternatively, continue reading the documentation to learn more about the protocol and its smart contracts.

### Security audits

The staking contracts have been reviewed and audited by the Ottersec and Halborn security teams. Once the reports are available, we will publish them here.

_\*In Aptos, the term ‘coin’ is used to denote both what is called ‘coins’ and ‘tokens’ in other blockchain ecosystems. Thus, we use ‘LP coins’ here as the correct term in the context of the Aptos architecture, though the meaning is the same as in ‘LP tokens’ on other chains._
