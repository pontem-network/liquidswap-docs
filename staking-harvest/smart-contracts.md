# Smart Contracts

Repository: [https://github.com/pontem-network/harvest](https://github.com/pontem-network/harvest/tree/main)

The harvest repository contains the main staking code core, entry functions, and different tests.

### Branches and Versions

Release tags are to be used for deployment on production (mainnet).

You can always find the latest release in the [Releases](https://github.com/pontem-network/harvest/releases) section and use it for the relevant dependencies in your project.&#x20;

### Addresses & Networks

The deployed contracts are on both mainnet and testnet.&#x20;

The **mainnet** address used to deploy is as follows:

```
0xbaaf53f7b9017ec4be9623c60b6182f2ffba1f01bc67f292295bcfe17f327701
```

The **testnet** addresses can be found in the following [PR](https://github.com/pontem-network/harvest/pull/38).&#x20;

### Staking contract

Source code: [./sources/stake.move](https://github.com/pontem-network/harvest/blob/main/sources/stake.move)

It's the core contract of the Liquidswap staking protocol. It manages staking pools, stakes, and reward distribution, and contains a rich list of getters.

The key part of the staking contracts is the `StakingPool<S, R>` resource, which contains all the information about a pool, rewards, events, and stakers:

```rust
// Stake pool, stores stake, reward coins and related info.
struct StakePool<phantom S, phantom R> has key {
   reward_per_sec: u64,
   // pool reward ((reward_per_sec * time) / total_staked) + accum_reward (previous period)
   accum_reward: u128,
   // last accum_reward update time
   last_updated: u64,
   // start timestamp.
   start_timestamp: u64,
   // when harvest will be finished.
   end_timestamp: u64,

   stakes: table::Table<address, UserStake>,
   stake_coins: Coin<S>,
   reward_coins: Coin<R>,
   // multiplier to handle decimals
   scale: u128,

   total_boosted: u128,

   /// This field can contain pool boost configuration.
   /// Pool creator can give ability for users to increase their stake profitability
   /// by staking nft's from specified collection.
   nft_boost_config: Option<NFTBoostConfig>,

   /// This field set to `true` only in case of emergency:
   /// * only `emergency_unstake()` operation is available in the state of emergency
   emergency_locked: bool,

   stake_events: EventHandle<StakeEvent>,
   unstake_events: EventHandle<UnstakeEvent>,
   deposit_events: EventHandle<DepositRewardEvent>,
   harvest_events: EventHandle<HarvestEvent>,
   boost_events: EventHandle<BoostEvent>,
   remove_boost_events: EventHandle<RemoveBoostEvent>,
}
```

* `phantom S` generic: the coin used as the staking asset
* `phantom R` generic: coin used the reward
* Reward amount distributed per second
* Some of the technical fields contain information about accumulated rewards, total stakes, and the coins' decimal scales
* Timestamps for the point when the harvest period started and when it ends
* Optional NFT boost configuration and the total boosted staking amount
* User's staking positions in the table
* Events
* Local emergency switch (paused while there is only one pool)

New staking pools are created on the registrar account, and the corresponding resource is placed in that account's storage. The registrar account interacts with the pools created on that account, so it's important. The pools on the same account can't be the same.

Liquidswap uses a front-end interface with a list of pre-filled staking pools to avoid confusion.

#### Generics

Almost all functions in the staking module use two generics, `S` and `R`, as already mentioned:

* `phantom S` generic - coin used to be a stake.
* `phantom R` generic - coin used to be a reward.

So for a staking pool that allows staking Liquidswap LPs for the APT/USDC (LayerZero) pair on the mainnet, the `S` generic will look like this:

```rust
0x05a97986a9d031c4567e15b797be516910cfcb4156312482efc6a19c0a30c948::lp_coin::LP<
    0xf22bede237a07e121b56d91a491eb7bcdfd1f5907926a9e58338f964a01b17fa::asset::USDC,
    0x1::aptos_coin::AptosCoin,
    0x190d44266241744264b964a37b8f09863167a12d3e70cda39376cfb4e3561e12::curves::Uncorrelated
>
```

And the `R` generic will be:

```rust
0x1::aptos_coin::AptosCoin
```

To make this work with your Move smart contract, see [Integration Examples](integration-examples.md).

#### Common logic

It's important to mention that accumulated rewards, user-earned rewards, and other technical variables are updated each time a user stakes, unstakes, harvests, or boosts/removes a boost.

This way, we don't need to recalculate the rewards for a specific user; we can execute the required mathematical operations at once for everyone, which makes the contract less expensive and as functional as possible.

#### NFT config

The NFT config is presented as the following resource and is optional for a staking pool. However, if a config is provided, users can boost their staking rewards using NFTs from a specified collection.

The config itself looks like this:

```rust
struct NFTBoostConfig has store {
    boost_percent: u128,
    collection_owner: address,
    collection_name: String,
}
```

* Contains the collection's creator/owner and name to determine the collection for the pool.
* Stake Boost percentage: should be between 1 and 100.

To create a config, you can use the function `stake::create_boost_config`.

#### Functions

In most cases, you can use [Scripts](smart-contracts.md#scripts) entry functions, but if you want to work directly with the staking core, the following would be helpful.

Almost all functions require the address of the pool (`pool_addr`) and the address of the account that created the pool:

* `create_boost_config` - creates and returns a new boost NFT config used in the `register_pool` function.
* `register_pool<S, R>` - creates a new pool on the owner address; also requires the values for rewards and duration. The boost config `nft_boost_config` is optional.
* `deposit_reward_coins<S, R>` - allows to deposit more rewards in the pool, requires rewards coins, extends harvest periods.
* `stake<S, R>` - stakes user coins. If a user stake already exists, it updates the current stake and relocks it again for one week.
* `unstake<S, R>` - unstakes the coins staked by the user; requires a value for the amount to unstake and returns the value for unstaked coins.
* `harvest<S, R>` - harvests rewards for user stake and returns reward coin.
* `boost<S, R>` - bosts user stake with the provided NFT; requires `Token` as the argument.
* `remove_boost<S, R>` - removes the NFT boost from the user stake and returns `Token`.&#x20;

**Getters**

* `get_boost_config<S, R>` - get the NFT boost configuration parameters: collection creator, name, boost percentage.
* `is_finished<S, R>` - get true if the harvest period is finished.
* `get_end_timestamp<S, R>` - get the end timestamp of the harvest period.
* `pool_exists<S, R>`  - get true if the pool exists.
* `stake_exists<S, R>` - get true if the user stake exists.
* `get_pool_total_stake<S, R>` - returns the total staked amount for the pool.
* `get_pool_total_boosted<S, R>` - returns the total boosted stake amount for the pool.
* `get_user_stake<S, R>` - get the user stake amount in the pool.
* `get_user_boosted<S, R>` - get the user's boosted stake amount in the pool.
* `get_pending_user_rewards<S, R>` - get the amount the user's rewards that are pending (awaiting harvest).
* `is_emergency` / `is_local_emergency` - get true if a global or local emergency happens.
* `is_boostable` - get true if the pool stake can be boosted with an NFT.
* `get_start_timestamp` - get the timestamp for when the pool was created.
* `is_boosted` - get true if the user stake is boosted.
* `get_unlock_time` - get the timestamp for the point when a user stake can be unstaked.
* `is_unlocked` - get true if a user stake can be unstaked.

The rest of the functions are related to emergency or admin accounts. If you'd like a more detailed explanation, check the detailed comments in the[ source code](https://github.com/pontem-network/harvest/blob/main/sources/stake.move) section.

### Scripts

Source code: [./sources/scripts.move](https://github.com/pontem-network/harvest/blob/main/sources/scripts.move)

The scripts module is an entry function that can be called diretly in a transaction and used by the Liquidswap dApp.

It makes the development task easier by hiding most of the complexity. Also, it works directly with user balances/NFTs while the core accepts NFTs and coins as part of the arguments.

#### Functions

* `register_pool<S, R>` - creates a new pool without an NFT boost config and extracts reward coins from the user account.
* `register_pool_with_collection<S, R>` - creates a new pool with an NFT boost config enabled; extracts reward coins from the user account.
* `stake<S, R>` - stakes the user's coins and extracts stake coins from the user account.
* `stake_and_boost<S, R>` - stakes the user's coins and boosts them with the provided NFT. Both the staking coins and the NFT are extracted from the user account.
* `unstake<S, R>` - unstakes the user's coins and deposits them back into the account.
* `unstake_and_remove_boost<S, R>` - unstakes the user's coins and the NFT boost, deposits both the coins and the NFT back on the account.
* `harvest<S, R>` - harvests user rewards and deposits them on the account.
* `deposit_rewards_coins<S, R>` - extract rewards from the user account and deposit them into the pool: extends the pool's duration.
* `boost<S, R>` - boosts the user's stakes with an NFT, extracting the NFT from the user account.
* `remove_boost<S, R>` - removes the NFT boost from the user's staking positions, depositing the NFT back to the user account.

### Config

Source code: [./sources/stake\_config.move](https://github.com/pontem-network/harvest/blob/main/sources/stake\_config.move)

This is in case of a global emergency: if it occurs, all the pools will be stopped. We don't believe this will ever happen, but we still want to cover the possible extreme cases.

Also, the contracts allow changing the emergency and treasury admin accounts.

