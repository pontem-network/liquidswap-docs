# Smart Contracts

Repository - [https://github.com/pontem-network/harvest](https://github.com/pontem-network/harvest/tree/main)

The harvest repository contains the main staking code core, entry functions, and different tests.

### Branches and Versions

The current `main` branch is the development branch and always contains the latest changes.

Release tags used for deployment on production (mainnet) in the future. You can always find the latest release in the [Releases](https://github.com/pontem-network/harvest/releases) section and use it for dependency in your project.&#x20;

### Addresses & Networks

The contracts deployed on testnet. The address used to deploy:

```
0xbaaf53f7b9017ec4be9623c60b6182f2ffba1f01bc67f292295bcfe17f327701
```

### Staking contract

Source code - [./sources/stake.move](https://github.com/pontem-network/harvest/blob/main/sources/stake.move)

It's the core contract of the Liquidswap staking protocol. It manages to stake pools, stakes, and rewards distribution and contains a rich list of getters.

The main part of the staking contracts is `StakingPool<S, R>` resource, which contains all information about the pool, rewards, events and stakers:

```rust
/// Stake pool, stores stake, reward coins and related info.
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
    stake_scale: u128,

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

* `phantom S` generic - coin used to be a stake.
* `phantom R` generic - coin used to be a reward.
* Amount of rewards distributed per second.
* Some technical fields contain information about an accumulated reward, total stakes, and coins' decimal scales.
* Timestamp when the harvest period ends.
* Optional NFT boost configuration and total boosted stakes amount.
* Users stakes in the table.
* Events.
* Local emergency switch (when only one pool paused).

New staking pools are created on the registrar account, and the corresponding resource is placed in that account's storage. The registrar account interacts with pools created on that account, so it's important. Pools on one account can't be the same.

Liquidswap uses a front-end interface with a list of pre-filled staking pools to avoid confusion.

#### Generics

Almost all functions in the staking module use two generics, `S` and `R`, as already mentioned:

* `phantom S` generic - coin used to be a stake.
* `phantom R` generic - coin used to be a reward.

So for the staking pool, which allows staking LP of Liquidswap pair APT/USDC (LayerZero) on the mainnet, the `S` generic will be:

```rust
0x05a97986a9d031c4567e15b797be516910cfcb4156312482efc6a19c0a30c948::lp_coin::LP<
    0xf22bede237a07e121b56d91a491eb7bcdfd1f5907926a9e58338f964a01b17fa::asset::USDC,
    0x1::aptos_coin::AptosCoin
>
```

And the `R` generic will be:

```rust
0x1::aptos_coin::AptosCoin
```

To make it work in your Move smart contract, see [Integration Examples](integration-examples.md).

#### Common logic

Important to mention that accumulated rewards, user-earned rewards, and other technical variables update each time users stake, unstakes, harvest, boost/remove boost.

In that way, we wouldn't need to recalculate rewards for a specific user; we can do mathematical operations at once for everyone, which makes the contract less expensive and possibly functional.

#### NFT config

The NFT config is presented as the following resource and can be optional for the staking pool. However, if the config is provided - users can boost their stake using NFT from the defined collection.

The config itself looks like this:

```rust
struct NFTBoostConfig has store {
    boost_percent: u128,
    collection_owner: address,
    collection_name: String,
}
```

* Contains collection creator/owner and name to determine collection for the pool.
* Stake Boost percent - between 1 and 100.

To create a config, you can use the function `stake::create_boost_config`.

#### Functions

For most cases, you can use [Scripts](smart-contracts.md#scripts) entry functions, but if you want to work directly with the staking core, the following would be helpful.

Almost all functions require the address of the pool (`pool_addr`), the address of the account which created the pool:

* `create_boost_config` - creates and returns a new boost NFT config used in the `register_pool` function.
* `register_pool<S, R>` - creates a new pool on the owner address, also requires providing rewards and duration. Boost config `nft_boost_config` is optional.
* `deposit_reward_coins<S, R>` - allows to deposit more rewards in the pool, requires rewards coins, extends harvest periods.
* `stake<S, R>` - stakes user coins. If a user stake exists, it updates the current stake and relocks it again for one week.
* `unstake<S, R>` - unstakes user-staked coins, requires an amount to unstake, and returns unstaked coins.
* `harvest<S, R>` - harvests rewards for user stake and returns reward coin.
* `boost<S, R>` - bosts user stake with NFT, requires `Token` as an argument.
* `remove_boost<S, R>` - removes NFT boost from user stake, returns `Token`.&#x20;

**Getters**

* `get_boost_config<S, R>` - get NFT boost configuration parameters: collection creator, name, boost percent.
* `is_finished<S, R>` - get true if the harvest period is finished.
* `get_end_timestamp<S, R>` - get the end timestamp of the harvest period.
* `pool_exists<S, R>`  - get true if pool exists.
* `stake_exists<S, R>` - get true if user stake exists.
* `get_pool_total_stake<S, R>` - returns total staked amount of the pool.
* `get_pool_total_stake<S, R>` - returns the total staked amount of the pool.
* `get_user_stake<S, R>` - get the user stake amount in the pool.
* `get_user_boosted<S, R>` - get user boosted stake amount in the pool.
* `get_pending_user_rewards<S, R>` - get the amount of user rewards that pending harvest.
* `is_emergency` / `is_local_emergency` - get true if a global or local emergency happens.
* `is_boostable` - get true if the pool stake can be boosted with NFT.
* `get_start_timestamp` - get timestamp of a pool creation.
* `is_boosted` - get true if user stake boosted.
* `get_unlock_time` - get timestamp when a user stake can be unstaked.
* `is_unlocked` - get true if a user stake can be unstaked.

The rest of the functions are related to emergency or admin accounts. If you are interested in a more detailed explanation, look at [the source code](https://github.com/pontem-network/harvest/blob/main/sources/stake.move), it has rich comments.

### Scripts

Source code - [./sources/scripts.move](https://github.com/pontem-network/harvest/blob/main/sources/scripts.move)

The scripts module is an entry function that can be called in a transaction directly and used by Liquidswap DApp.

It makes work simple by hiding most of the complexity. Also, it works directly with user balances/NFTs while core accepting NFTs and coins as part of arguments.

#### Functions

* `register_pool<S, R>` - creates a new pool without NFT boost config and extracts rewards coins from the user account.
* `register_pool_with_collection<S, R>` - creates a new pool with NFT boost config enabled extracts rewards coins from the user account.
* `stake<S, R>` - stakes user's coins, extracts stake coin from user account.
* `stake_and_boost<S, R>` - stakes the user's coins and boost it with provided NFT. Both stake coins and NFT are extracted from the user account.
* `unstake<S, R>` - unstakes user coins and deposit back into the account.
* `unstake_and_remove_boost<S, R>` - unstakes user coins and NFT boost, deposits coins and NFT back on the account.
* `harvest<S, R>` - harvests user rewards and deposits them on the account.
* `deposit_rewards_coins<S, R>` - extract rewards from the user account and deposit it to the pool: extend pool duration.
* `boost<S, R>` - boost user stakes with NFT, extracts NFT from the user account.
* `remove_boost<S, R>` - remove boost from user stakes with NFT, deposit NFT back to the user account.

### Config

Source code - [./sources/stake\_config.move](https://github.com/pontem-network/harvest/blob/main/sources/stake\_config.move)

Controls global emergency - in that case, will stop all pools. We think it never happened, but we still want to cover possible extreme cases.

Also, the contracts allow changing admin accounts: emergency and treasury.

