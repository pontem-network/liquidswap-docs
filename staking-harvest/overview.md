# Overview

The Liquidswap staking allows the creation of new staking pools where the creator configures stake and reward coins types and the duration of the harvest period.

The staking pool stores all information regarding staking itself, such as rewards, stakes, duration, etc. The staking pool will be created and stored on the creator account of the pool.

At the stage of creation, the pool would calculate the number of rewards that can be distributed per second using the following formula:

`reward_per_second = rewards / duration`&#x20;

The accumulated rewards since the last harvest event would be split between all stackers during the harvest. The accumulated reward will go to the [treasury](overview.md#treasury) if there is no stacker.

In the first step accumulated reward is calculated and divided by the total stake:

`accum_reward = (reward_per_sec * time_passed) / total_stake`

In the second step, we calculate the user reward:

`user_reward = (accum_reward * user_stake) - unobtainable_reward`

The unobtainable reward is used to fix rewards that accumulated before the user joined the pool and doesn't allow to take additional rewards in specific cases.

### Week Lock

All stakers are locked for one week by default. After that one-week stakers can unlock their stake. If the harvest period finishes early, it's possible to unlock early.

### Harvest

To earn rewards, users should harvest on their stake - send a transaction to call the harvest function from time to time. It can be done with the DApp user interface.

### Extending of harvest period

Deposit more rewards to the pool is possible after pool creation. Anyone with a rewards coin can do it before the end of the harvest period.

By depositing more rewards to the pool, the harvest period will be extended on time got from the formula:

`extend_seconds = extended_rewards / rewards_per_sec`

So, if you want to extend the harvest period, deposit more rewards into the pool.

### After harvest period

After the harvest period (when the end timestamp is reached), it is still possible for a user to harvest his remaining rewards and unstake his coins/NFT. Yet, accumulating new rewards, staking more, or extending the pool duration wouldn't be possible.

### Stake Coin & Rewards Coin

Any coin in the Aptos blockchain (indeed registered one) can be used as a reward coin or stake coin, and both reward/stake can be the same coin.&#x20;

**Important:** It may not work with exotic coins (large decimals amounts, too large supply), so use at your own risk and double-check (e.g., by forking and running tests with your coin, etc.). Also, very important to be sure that the amount of the reward (including the amount that will be distributed per second) will be enough to distribute between all possible stakers :warning:

### NFTs stake

NFT collection can be configured during the creation of the pool, and NFTs from the collection can be a stake in addition to the stake itself and boost stake power.

Each pool configured for NFT boost has a boost percent and user stakes becoming boosted by the formula:

```
boosted_stake = stake * boost_percent / 100
```

### Treasury

The treasury account is a multisignature account created with [Momentum Safe](https://github.com/Momentum-Safe).

The treasury withdraws the remaining rewards three months after the harvest period is finished or immediately in an emergency.

Later treasury will be transferred under the control of the DAO account.

### Emergency

The staking contracts can stop in an emergency, while an emergency account controls the emergency mechanism. The account is a multisignature account created with [Momentum Safe](https://github.com/Momentum-Safe).

Contracts have two kinds of emergency: by the pool and global. In the case of global, all pools would be stopped by emergency. Otherwise, it can stop only a specific pool. Emergency can't be undone, as it breaks staking math.

In an emergency, stakers can withdraw their stake (coins/NFT) immediately, and the treasury account can withdraw the remaining rewards.

We will transfer the emergency role under the control of the DAO account later.
