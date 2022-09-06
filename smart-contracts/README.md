# Smart Contracts

The contracts itself published in the Liquidswap GitHub repository - [https://github.com/pontem-network/liquidswap](https://github.com/pontem-network/liquidswap).

The Liquidswap is built from smart contracts written in Move language executed by Move VM. The contracts are separated on core logic with safety guarantees and periphery contracts, allowing for a well-tested reasonable small core layer and many versions of the periphery that can work with core utilizing their custom logic.

For example, we provide the basic `Router.move`, which is used by our frontend, proxying all Liquidity Pool module methods with additional safe and price checks, while it can't be a way for traders who want to save as much gas as possible. Such traders or organizations can write their custom routers or call the core contracts directly.

## Branches and versions

The current `main` branch is the development branch and contains the latest changes always.

Release branches contain the latest changes indeed created for that release.

Current `release-v0.2` and release tags `v0.2*` are going to the mainnet release.

Some features from the `main` branch can be missed for the new releases like flash loans probably would come available only in `v0.3.*`.

You can always see the latest release on the [Releases](https://github.com/pontem-network/liquidswap/releases).

## Integrations

This document explains how Liquidswap contracts are organized at the top level.

But if you are already familiar with it, you can go to [integrations](../integration/) docs.

## Liquidity Pool

Source Code: [./sources/swap/liquidity\_pool.move](https://github.com/pontem-network/liquidswap/blob/main/sources/swap/liquidity\_pool.move).

The Liquidity Pool module implements all swap logic, LP logic, math (constant product formula), and core checks.

The core thing of the Liquidity Pool contract is a resource that describes liquidity pools themselves and holds all needed information:

{% code lineNumbers="true" %}
```rust
struct LiquidityPool<phantom X, phantom Y, phantom LP> has key {
    coin_x_reserve: Coin<X>,
    coin_y_reserve: Coin<Y>,
    last_block_timestamp: u64,
    last_price_x_cumulative: u128,
    last_price_y_cumulative: u128,
    lp_mint_cap: coin::MintCapability<LP>, 
    lp_burn_cap: coin::BurnCapability<LP>,
    x_scale: u64,
    y_scale: u64,
    curve_type: u8,
    locked: bool,
}
```
{% endcode %}

The resource holds all important information, including:

* Reserves for both coins `X` and `Y`.
* Cumulative price information.
* Mint and burn capabilities for `LP` coins.
* Scales of coins decimals (used for stable swaps).
* Type of curve.
* And if pool locked (used during flash loans).

The new liquidity pools are created on the registrar account, and the resource will be stored in that account storage.

Unlike Uniswap, for Liquidswap, there could be many pools with the same coins inside but distributed around many addresses.

To don't confuse, Liquidswap uses frontend with pre-filled pools and our pools registry to show users mostly verified pools.

### Generics

A set of `(X, Y, LP, pool_address)` uniquely identifies a LiquidityPool on the blockchain.

Because of that, all functions that act on pools have three generic parameters and `pool_addr: address` as a parameter, for example:

```move
public entry fun swap<X, Y, LP>(... pool_addr: address, ...) {} 
```

* `X, Y` - types of coins to be swapped in the pool,
  for example, `0x1::aptos_coin::AptosCoin` and `liquidswap_lp::coins::USDT`(wrapped USDT)

* `LP` - type of LP coin for the pool, explained below

* `pool_address` - pool owner address

Note: **Generics must be sorted**, see [Coins Sorting](./#coins-sorting).

### Functions

**In most cases, you don't need to use the `LiquidityPool` module itself.
`Router` module provides high-level wrappers around `LiquidityPool`, which simplify most of the tasks.**

Operations with liquidity:

* `register<X, Y, LP>` - creates new liquidity pool on an account. The pool will be stored under the registrar account address.
* `mint<X, Y, LP>` - mints new LP coins in exchange for `Coin<X>` and `Coin<Y>`.
* `burn<X, Y, LP>` - burns some LP coins in exchange for `Coin<X>` and `Coin<Y>`.
* `swap<X, Y, LP>` - swaps `Coin<X>` or `Coin<Y>` or both and get `Coin<X>` or `Coin<Y>` or both in exchange.

Getters:

* `is_pool_locked<X, Y, LP>` - returns bool determines if pool locked (flashloaned).
* `get_reserves_size<X, Y, LP>` - returns current reserves size for both `Coin<X>` and `Coin<Y>`.
* `get_curve_type<X, Y, LP>` - returns curve type for the pool.
* `get_decimals_scales<X, Y, LP>` - returns decimals scales for both `Coin<X>` and `Coin<Y>,` would return correct values only for stable pools.
* `pool_exists_at<X, Y, LP>` - returns bool determines if pool exists.
* `get_fees_config<X, Y, LP>` - returns fees config for the pool.
* `get_cumulative_prices<X, Y, LP>` - return cumulative prices and last block timestamp.

#### Coins Sorting

Unlike the [Router](./#router), if you want to work directly with Liquidity Pool functions, you have to sort coins that you pass as generics.

Sorting is important as it brings rules for creating liquidity pools and, how coins must be sorted, what's successfully used in `Router.`

The current sorting algorithm takes coins symbols, convert them to bytes using `BCS,` and compares them byte per byte (starting from the end).

* You can look at the implementation in Move language in the Coin Helper module - [coin\_helper.move](https://github.com/pontem-network/liquidswap/blob/main/sources/libs/coin\_helper.move#L44).
* Implementation in Javascript - [Coins Sorting In JS](https://gist.github.com/borispovod/7c81bf5d82dbae1c26b95ff8b6861d4a).
* Also, it supported in [Typescript SDK](../typescript-sdk.md).

### Curve types

During the creation of the pool, the curve type can be provided; it's simple, as an integer used to determine which type of pool must be created:

* `1` - Stable curve type.
* `2` - Uncorrelated curve type.

You can read more about curve formulas at the [Protocol Overview](../protocol-overview.md) page.

### LP coins

The `LP` coin for pairs `X` and `Y` can be any type not initialized as a coin yet.

So, you can always deploy your LP coin module, which contains a type and functions your need if you need, and pass it as a third generic to the `register<X, Y, LP>` function.

Like this [LP module](https://github.com/pontem-network/liquidswap-lp/blob/main/sources/lp.move#L2) by Pontem can be used for many other teams who were creating their liquidity pools:

```
/// Basic liquidity coin, can be pre-deployed for any account.
module liquidswap_lp::lp {
    /// Represents `LP` coin with `X` and `Y` coin types.
    struct LP<phantom X, phantom Y> {}
}
```

If you don't want to use too many generics, you can deploy a new specific type for each pool, like:

```
module my_account::lp {
    struct LP_USDT_APTOS {}
}
```

And use the provided type to register your pool for the `USDT` and `APTOS` pair.

## Router

Source code: [./sources/swap/router.move](https://github.com/pontem-network/liquidswap/blob/main/sources/swap/router.move).

The Router module is a periphery layer on top of the Liquidity Pool module.

The Router has additional checks that verify the amounts the developer/trader/liquidity provider wants to exchange/mint/burn are reasonable.

Also, it sorts coins automatically and has several useful getters that allow you to estimate exchange prices.

In most cases, we recommend using a Router if you want to work with Liquidity Pool.

There can be many routers from different teams, yet Liquidswap standard Router should be enough for most cases, and it's audited.

### Functions

The router functions accept `Coin<X>` and `Coin<Y>`, `Coin<LP>` resource from aptos\_framework::coin so that they can be easily integrated with 3rd party module. But it can't be called directly from the transaction; if you need entry points, look at Scripts.

Developers iterating with functions can order generics however they want. Let's say if I want to swap `aptos_framework::aptos_coin::AptosCoin` to `liqudswap_lp::coins::USDT` i can just use the function:

```
router::swap_exact_coin_for_coin<
    aptos_framework::aptos_coin::AptosCoin,
    liqudswap_lp::coins::USDT,
    ...
>
```

Or if i can do it vice-versa (`USDT` -> `APTOS`), i can just reorder generics:

```
router::swap_exact_coin_for_coin<
    liqudswap_lp::coins::USDT,
    aptos_framework::aptos_coin::AptosCoin,
    ...
>
```

Liquidity operations:

* `register_pool<X, Y, LP>` - register new pool with `Coin<X>` and `Coin<Y>` as reserves, and `LP` as lp coin.
* `add_liquidity` - add liquidity (`Coin<X>` and `Coin<Y>`) to existing pool.
* `remove_liquidity` - burn `Coin<LP>` and get `Coin<X>` and `Coin<Y>` back with checks.
* `swap_exact_coin_for_coin` - swap exact amount of `Coin<X>` to get at least minimum amount of `Coin<Y>`.
* `swap_coin_for_exact_coin` - swap maximum amount of `Coin<X>` to get exact amount of `Coin<Y>`.
* `swap_coin_for_coin_unchecked` - just swaps coins without any checks.

Getters:

* `get_amount_out<X, Y, LP>` - estimate `Y` coins amount out from swap of `X` coins amount in. Can eat a lot of gas for stable pools.
* `get_amount_in<X, Y, LP>` - estimate amount of `X` coins needed to get amount out of `Y` coins.
* `calc_optimal_coin_values<X, Y, LP>` - calculates optimal amount of `Coin<X>` and `Coin<Y>` you need to add liquidity and get fair return in LP coins.
* `get_reserves_size<X, Y, LP>` - returns reserves for both coins X and coin Y which holds by the pool.
* `pool_exists_at<X, Y, LP>` - returns bool determines if pools exists.
* `get_decimals_scales<X, Y, LP>` - returns decimals scales of the pool, returns correct values only for stable pools.
* `get_cumulative_prices<X, Y, LP>` - returns cummulative prices and last block timestamp.
* `get_curve_type<X, Y, LP>` - returns curve type of the pool.
* `get_reserves_for_lp_coins<X, Y, LP>` - returns amount of `Coin<X>` and `Coin<Y>` user gets after burning liquidity.

## Scripts

Source code: [./sources/swap/scripts.move](https://github.com/pontem-network/liquidswap/blob/main/sources/swap/scripts.move).

The top-level module contains entry functions, which users can execute directly by sending transactions, or 3rd party modules can use by utilizing `signer`.

Generalized summary:

* Requires signer.
* Accepting numbers as arguments representing coins values.
* Extracting coins directly from the signer account.
* Registering LP coin on account if it's not registered.
* Works with default [Router](./#router).

Perfect way to iteract with Liquidswap if you want to call swap from CLI or UI using standard router.

### Functions

All functions has "slippage" amount arguments, like `coin_x_val_min`, `coin_y_val_min` or `min_x_out_val`, etc.

* `register_pool<X, Y, LP>` - register pool with curve type.
* `register_pool_and_add_liquidity<X, Y, LP>` - register pool with curve type and add liquidity immidiatelly (extracted from signer balance), LP coins deposited to signer.
* `add_liquidity<X, Y, LP>` - add liquidity to existing pool, LP coins would be deposited to signer.
* `swap<X, Y, LP>` - swap `Coin<X>` for `Coin<Y>`, developer has to provide amount of `X` coins `coin_val` to get at minimum `coin_out_min_val` of Y coins.
* `swap_into<X, Y, LP>` - swap maximum amount of `Coin<X>` for exact coin `Coin<Y>`. The remainder of `Coin<X>` will be deposited back to account.

## Helpers

Source Code: [.sources/libs](https://github.com/pontem-network/liquidswap/tree/main/sources/libs)

The helpers library allowed us to make code more readable and extract parts of math, work with coins and many other things to seperated files.

### Coin Helper

Source Code: [.sources/libs/coin\_helper.move](https://github.com/pontem-network/liquidswap/tree/main/sources/libs/coin\_helper.move)

Mostly helpers on top of `aptos_framework::coin`.

#### Functions

* `assert_is_coin<CoinType>` - aborts if provided `CoinType` is not a coin.
* `compare<X, Y>` - comparing `X` coin symbols and `Y` coin symbols.
* `is_sorted<X, Y>` - returns bool determines `X` and `Y` coins are sorted.
* `supply<CoinType>` - used to extract supply from `LP`, ignoring `Option`.
* `generate_lp_name<X, Y>` - generates name and symbol for `LP` coin from `X` and `Y` coins symbolks, used in default router, not mandatory.

### Math

Source Code: [.sources/libs/math.move](https://github.com/pontem-network/liquidswap/tree/main/sources/libs/math.move)

Implements basic math helpers.

#### Functions

* `overflow_add` - adding two `u128` and just overflow numbers if needed without abort.
* `mul_div` - `x * y / z` for `u64` numbers.
* `mul_div_u128` - `x * y / z` for `u128` numbers but returns `u64`.
* `mul_to_u128` - muls two `u64` to `u128`.
* `sqrt` - get square root using Babylonian method.
* `pow_10` - returns `10^degree`.

### Deps libraries

The smart contracs utilizing math libraries created by Pontem team:

* [u256](https://github.com/pontem-network/u256) - pure Move `u256` implementation because some numbers aren't fit in `u128`.
* [uq64x64](https://github.com/pontem-network/uq64x64) - Q number format for cumulative price calculations.

### Advanced topics

#### Flashloans

The flashloans implemented in `main` branch as concept currently and at least for now we are not planning to launch it on first day.

It's based on Move "loan" concept, when during flashloan the developer get in return Move object contains loan data and that can't be stored, copied, cloned or dopped, so only what left for developer - return the object back to `pay_flashloan` function, which verifying constant product function in result.

The concept explained early by Pontem team in the [medium article](https://medium.com/p/bbc48a48d93c).

#### VE(3,3)

The `ve(3,3)` logic is currently under active development and even could be significant changed.

#### Oracle

The oracle concept based on cumulative prices algorithm from Uniswap v2. This why prices can overflow (similar how it's original done).

The fields in Liquidity Pool resource storing prices:

```
last_block_timestamp: u64,
last_price_x_cumulative: u128,
last_price_y_cumulative: u128,
```

And the function of Liquidity Pool module `get_cumulative_prices<X, Y, LP>` allows to extract prices from the pool.

If you are interesting in Oracle implementation look at our [integration](../integration/) guides.

#### Treasury

Source Code: [.sources/swap/dao\_storage.move](https://github.com/pontem-network/liquidswap/blob/main/sources/swap/dao\_storage.move)

DAO Treasury contract brings to each liquidity pool ever created on the protocol and it's collecting 0.1% of all fees on the protocol.

Managed by treasury admin which should be multisig, later will be updated with full-fledged Pontem DAO.

#### Emergency

Source Code: [.sources/swap/emergency.move](https://github.com/pontem-network/liquidswap/blob/main/sources/swap/emergency.move)

Allows to pause/resume all swaps by emergency account.

The contract itself can be disabled forever in case we see swap is stable and everything is smooth.
