# Overview

Liquidswap staking allows developers to create new staking pools, configuring the stake and reward coin types and the duration of the harvest period.

A staking pool stores all the information regarding staking itself, such as rewards, stakes, duration, etc. Any new staking pool will be created and stored on the pool creator's account.

At the point of creation, the pool will calculate the number of reward coins that can be distributed per second using the following formula:

`reward_per_second = rewards / duration`&#x20;

All accumulated rewards will be split between all the stakers during harvesting. If there are no stakers, all accumulated rewards will go to the [treasury](overview.md#treasury).&#x20;

In the first step, the accumulated reward is calculated and divided by the total staked amount:

`accum_reward = (reward_per_sec * time_passed) / total_stake`

In the second step, we calculate the user's reward:

`user_reward = (accum_reward * user_stake) - unobtainable_reward`

The 'unobtainable reward' part is used to fix the rewards that were accumulated before the user joined the pool and prevents the user from claiming excessive rewards in specific cases.

### Weekly Lock

All stakes are locked for one week by default. After that initial week elapses, stakers can unlock their coins. If the harvest period finishes early, it's also possible to unlock early.

### Harvest

To earn rewards, users should perfrom harvesting from time to time by sending a transaction to call the harvest function from. It can be done via the dApp user interface.

### Extending the harvest period

Depositing more rewards into the pool becomes possible immediately after the pool is created. Anyone who holds reward coins can do it before the end of the harvest period.

Whenever additional rewards are deposited into the pool, the harvest period is extended by the amount of time calculated according to the formula:

`extend_seconds = extended_rewards / rewards_per_sec`

Therefore, if you want to extend the harvest period, simply deposit more rewards into the pool.

### After the harvest period

After the harvest period (once the end timestamp is reached), it is still possible for a user to harvest their remaining rewards and unstake their coins/NFT. Yet, it will not be possible for them to accumulate new rewards, stake more, or extend the pool duration.

### Staking Coin & Reward Coin

Any coin issued on the Aptos blockchain (and registered) can be used as the reward coin or staking coin, and both the rewards and stakes can be denominated in the same coin.

Decimals from the Staking coin and Rewards coin are used to calculate scale. The scale is a variable used to get quite accurate results for rewards, share:

```
scale = (10e12 / pow(10, reward_decimals)) * pow(10, stake_decimals)
```

**Important:** It may not work with exotic coins (large decimal numbers, extremely large supply), so use this feature at your own risk and double-check that everything works (e.g. by forking and running tests with your coin, etc.). Also, it is very important to be sure that the reward amount (including the amount that will be distributed per second) is enough to be distributed among all the potential stakers :warning:

### NFT staking

The NFT collection for boosting can be configured during the pool's creation, and NFTs from that collection can be used as a stake in addition to the staking coins themselves to boost the staking power.

Each pool configured for an NFT boost has its own boost percentage rate, and user stakes are boosted according to the formula:

```
boosted_stake = stake * boost_percent / 100
```

### Treasury

The treasury account is a multisignature account created with [Momentum Safe](https://github.com/Momentum-Safe).

The treasury withdraws any rewards left three months after the harvest period is completed - or immediately in case of an emergency.

In the future, the treasury will be transferred under the control of the DAO account.

### Emergency

Staking contracts can be stopped in case of an emergency, while the emergency account controls the emergency mechanism. The account is a multisignature account created with [Momentum Safe](https://github.com/Momentum-Safe).

Contracts have two kinds of emergency coded into them: pool-specific and global. In case of a global event, all pools will be stopped. Otherwise, only a specific pool will be stopped. Emergency stoppage can't be undone, as it breaks the staking math.

In an emergency, stakers can withdraw their stakes (coins/NFT) immediately, and the treasury account can withdraw the remaining rewards.

We will transfer the emergency role under the control of the DAO account in the future.
