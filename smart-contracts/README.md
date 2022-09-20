# Smart Contracts

The contracts are published in the Liquidswap GitHub repository at [https://github.com/pontem-network/liquidswap](https://github.com/pontem-network/liquidswap).

Liquidswap uses smart contracts written in the Move language and executed by Move VM. The core contracts containing the protocol's logic and safety guarantees and are separated from the periphery contracts, allowing for a well-tested, reasonably small core layer and multiple versions of the periphery that can work with  the core utilizing their own custom logic.

For example, the basic `Router.move`, used by Liquidswap's frontend, proxys all Liquidity Pool module methods with additional safety and price checks. At the same time, traders who want to minimize gas usage can write their own custom routers or call the core contracts directly.

## Branches and versions

The current `main` branch is the development branch and always contains the latest changes.

Release branches contain the latest changes created for the specific release.

The current `release-v0.2` and release tags `v0.2*` are designed for the mainnet release.

Not all features from the `main` branch will neccesarily be present in the upcoming releases - for example, flash loans will probably become available only in `v0.3.*`.

You can always find the latest release in the [Releases](https://github.com/pontem-network/liquidswap/releases) section.

## Integrations

This document explains how Liquidswap contracts are organized at the top level.

If you are already familiar with this material, you can proceed to the [integrations](../integration/) docs.

## Liquidity Pool

Source Code: [./sources/swap/liquidity\_pool.move](https://github.com/pontem-network/liquidswap/blob/main/sources/swap/liquidity\_pool.move).

The Liquidity Pool module implements all swap logic, LP logic, math (constant product formula), and core checks.

The core part of the Liquidity Pool contract is a resource that describes the liquidity pools themselves and contains all the required information:

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

* Reserves of both tokens `X` and Y
* Cumulative price information
* Mint and burn capabilities for `LP` tokens
* Token decimal scales (used for stable swaps)
* Curve type
* If a pool is locked (needed for the flash loan feature)

New liquidity pools are created on the registrar account, and the corresponding resource is placed in that account's storage.

Unlike Uniswap, Liquidswap can have multiple pools for the same tokens but distributed across different addresses.

To avoid confusion, Liquidswap uses a frontend interface with a list of pools already pre-filled and a pools registry to show the users mostly verified pools.

### Generics

A set of `(X, Y, LP, pool_address)` uniquely identifies a liquidity pool on the blockchain.

Because of that, all functions that refer to pools have three generic parameters and `pool_addr: address` as a parameter, for example:

```
public entry fun swap<X, Y, LP>(... pool_addr: address, ...) {} 
```

* `X, Y` - the tokens to be swapped in a pool, for example, `0x1::aptos_coin::AptosCoin` and `liquidswap_lp::coins::USDT`(wrapped USDT)
* `LP` - type of the LP token corresponding to the pool, explained below
* `pool_address` - pool owner address

Note: **Generics must be sorted**, see [Coin Sorting](./#coins-sorting).

### Functions

**In most cases, you won't need to use the `LiquidityPool` module itself. The `Router` module provides high-level wrappers around `LiquidityPool`, which simplify most of the tasks.**

Operations with liquidity:

* `register<X, Y, LP>` - creates fora new liquidity pool on an account. The pool will be stored under the registrar account address.
* `mint<X, Y, LP>` - mints new LP tokens in exchange for `Coin<X>` and `Coin<Y>`.
* `burn<X, Y, LP>` - burns some LP tokens in exchange for `Coin<X>` and `Coin<Y>`.
* `swap<X, Y, LP>` - swaps `Coin<X>` or `Coin<Y>` or both to get `Coin<X>` or `Coin<Y>` or both in exchange.

Getters:

* `is_pool_locked<X, Y, LP>` - returns bool determining if the pool is locked (flashloaned).
* `get_reserves_size<X, Y, LP>` - returns the current reserves size for both `Coin<X>` and `Coin<Y>`.
* `get_curve_type<X, Y, LP>` - returns the curve type for the pool.
* `get_decimals_scales<X, Y, LP>` - returns the decimal scales for both `Coin<X>` and `Coin<Y> - but` correct values are returned only for stable pools.
* `pool_exists_at<X, Y, LP>` - returns bool determining if the pool exists.
* `get_fees_config<X, Y, LP>` - returns fees config for the pool.
* `get_cumulative_prices<X, Y, LP>` - return cumulative prices and the last block's timestamp.

#### Coin Sorting

Unlike the [Router](./#router), if you want to work directly with Liquidity Pool functions, you have to sort the tokens or coins that you pass on as generics.

Sorting is important, as it brings in the rules for creating liquidity pools and for how coins must be sorted that are successfully used in `Router.`

The current sorting algorithm captures token symbols, converts them into bytes using `BCS,` and compares them byte per byte (starting by comparing length and then symbol by symbol).

* Review the implementation in the Move language in the Coin Helper module: [coin\_helper.move](https://github.com/pontem-network/liquidswap/blob/main/sources/libs/coin\_helper.move#L44).
* Implementation in Javascript: [Coin Sorting In JS](https://gist.github.com/borispovod/7c81bf5d82dbae1c26b95ff8b6861d4a).
* Also, it's supported in [Typescript SDK](../typescript-sdk.md).

### Curve types

When creating a pool, the curve type can be provided; it's as simple as using an integer to determine the pool type:

* `1` - Stable curve type.
* `2` - Uncorrelated curve type.

You can read more about Liquidswap's curve formulas on the [Protocol Overview](../protocol-overview.md) page.

### LP tokens

The `LP` token for pairs `X` and `Y` can be of any type that isn't yet initialized as a coin/token.

Thus, you can always deploy your LP coin module, containiing the required type and functions, and pass it on as a third generic to the `register<X, Y, LP>` function.

The following [LP module](https://github.com/pontem-network/liquidswap-lp/blob/main/sources/lp.move#L2) by Pontem can be used by other teams creating liquidity pools:

```
/// Basic liquidity coin, can be pre-deployed for any account.
module liquidswap_lp::lp {
    /// Represents `LP` coin with `X` and `Y` coin types.
    struct LP<phantom X, phantom Y> {}
}
```

If you don't want to use too many generics, you can deploy a specific new type for each pool as follows:

```
module my_account::lp {
    struct LP_USDT_APTOS {}
}
```

Then use the specified type to register the pool for the `USDT` and `APTOS` pair.

## Router

Source code: [./sources/swap/router.move](https://github.com/pontem-network/liquidswap/blob/main/sources/swap/router.move).

The Router module is a periphery layer on top of the Liquidity Pool module.

The Router contains additional checks to verify that the amount that a developer, trader, or liquidity provider wants to exchange/mint/burn is reasonable.

Also, the module sorts tokens or coins automatically and has several useful getters that help estimating the swap price.

In most cases, we recommend using a Router if you want to work with the Liquidity Pool module.

Third-party teams can provide their own routers, but Liquidswap's standard Router should be enough for most cases. An added advantage is that it's already audited.

### Functions

The router functions accept the resources `Coin<X>,` `Coin<Y>`, `Coin<LP>` from aptos\_framework::coin so that they can be easily integrated with third-party modules. However, the resource cannot be called directly from a transaction; if you need entry points, refer to Scripts.

Developers interacting with functions can order generics in any way they wish. For example, if one wants to swap `aptos_framework::aptos_coin::AptosCoin` to `liqudswap_lp::coins::USDT, one`can just use the function:

```
router::swap_exact_coin_for_coin<
    aptos_framework::aptos_coin::AptosCoin,
    liqudswap_lp::coins::USDT,
    ...
>
```

A reverse swap (`USDT` -> `APTOS`) can be done by reordering the generics:

```
router::swap_exact_coin_for_coin<
    liqudswap_lp::coins::USDT,
    aptos_framework::aptos_coin::AptosCoin,
    ...
>
```

Liquidity operations:

* `register_pool<X, Y, LP>` - register a new pool with `Coin<X>` and `Coin<Y>` as reserves and `LP` as the liquidity provider token.
* `add_liquidity` - add liquidity (`Coin<X>` and `Coin<Y>`) to an existing pool.
* `remove_liquidity` - burn `Coin<LP>` and get `Coin<X>` and `Coin<Y>` back with checks.
* `swap_exact_coin_for_coin` - swap an exact amount of `Coin<X>` to get no less than the specified amount of `Coin<Y>`.
* `swap_coin_for_exact_coin` - swap no more than the specified maximum amount of `Coin<X>` to get an exact amount of `Coin<Y>`.
* `swap_coin_for_coin_unchecked` - simply swaps tokens without any checks.

Getters:

* `get_amount_out<X, Y, LP>` - estimate the amount of `Y` tokens resulting from a swap of a specified amount in`X` tokens. Can consume a lot of gas when used for stable pools.
* `get_amount_in<X, Y, LP>` - estimate the amount of `X` tokens needed to get a specified amount in`Y` tokens.
* `calc_optimal_coin_values<X, Y, LP>` - calculates the optimal amounts of `Coin<X>` and `Coin<Y>` needed to add liquidity and get a fair amount of LP tokens.
* `get_reserves_size<X, Y, LP>` - returns the reserves for both token X and token Y held by the pool.
* `pool_exists_at<X, Y, LP>` - returns bool determining if the pool exists.
* `get_decimals_scales<X, Y, LP>` - returns the pool's decimals scale. The resulting values will be correct only for stable pools.
* `get_cumulative_prices<X, Y, LP>` - returns cumulative prices and the last block's timestamp.
* `get_curve_type<X, Y, LP>` - returns the pool's curve type.
* `get_reserves_for_lp_coins<X, Y, LP>` - returns the amounts in `Coin<X>` and `Coin<Y>` that a user will receive after burning LP tokens.

## Scripts

Source code: [./sources/swap/scripts.move](https://github.com/pontem-network/liquidswap/blob/main/sources/swap/scripts.move).

The top-level module contains the entry functions that users can execute directly by sending transactions and that third-party modules can use via `signer`.

Summary:

* Requires a signer;
* Accepts numbers as arguments representing tokens' values;
* Extracts coins/tokens directly from the signer account;
* Registers a LP token on the account if it's not registered;
* Works with default [Router](./#router).

This is the optimal way to interact with Liquidswap if you want to call a swap from the CLI or the UI using the standard router.

### Functions

All the functions have 'slippage' amount arguments, like `coin_x_val_min`, `coin_y_val_min` or `min_x_out_val`, etc.

* `register_pool<X, Y, LP>` - register a pool with the specified curve type.
* `register_pool_and_add_liquidity<X, Y, LP>` - register a pool with the specified curve type and add liquidity immidiatelly (extracted from the signer's balance), with the resulting LP tokens deposited to the signer.
* `add_liquidity<X, Y, LP>` - add liquidity to an existing pool, with the resulting LP tokens deposited to the signer's address.
* `swap<X, Y, LP>` - swap `Coin<X>` for `Coin<Y>;` the developer has to provide the amount of `X` tokens/coins `coin_val` to get the minimum amount `coin_out_min_val` of Y.
* `swap_into<X, Y, LP>` - swap no more than the specified maximum amount of `Coin<X>` for an exact amount in `Coin<Y>`. The remainder of `Coin<X>` will be deposited back to the account.

## Helpers

Source Code: [.sources/libs](https://github.com/pontem-network/liquidswap/tree/main/sources/libs)

The helpers library improves code readability by extracting parts of the math, token interactions, and other things to separate files.

### Coin Helper

Source Code: [.sources/libs/coin\_helper.move](https://github.com/pontem-network/liquidswap/tree/main/sources/libs/coin\_helper.move)

Mostly helpers on top of `aptos_framework::coin`.

#### Functions

* `assert_is_coin<CoinType>` - aborts if the provided `CoinType` is not a coin/token.
* `compare<X, Y>` - compares `X` coin symbols and `Y` coin symbols.
* `is_sorted<X, Y>` - returns bool determining if `X` and `Y` tokens are sorted.
* `supply<CoinType>` - extracts supply from `LP`, ignoring `Option`.
* `generate_lp_name<X, Y>` - generates a name and symbol for the `LP` token from `X` and `Y` symbols. This is used in the default router and is not mandatory.

### Math

Source Code: [.sources/libs/math.move](https://github.com/pontem-network/liquidswap/tree/main/sources/libs/math.move)

Implements basic math helpers.

#### Functions

* `overflow_add` - adding two `u128` and just overflowing the numbers if needed without aborting.
* `mul_div` - `x * y / z` for `u64` numbers.
* `mul_div_u128` - `x * y / z` for `u128` numbers but returns `u64`.
* `mul_to_u128` - muls two `u64` to `u128`.
* `sqrt` - get the square root using the Babylonian method.
* `pow_10` - returns `10^degree`.

### Deps libraries

The smart contracs utilize the following math libraries created by the Pontem team:

* [u256](https://github.com/pontem-network/u256) - a pure Move `u256` implementation because some numbers don't fit in `u128`.
* [uq64x64](https://github.com/pontem-network/uq64x64) - the Q number format for cumulative price calculations.

### Advanced topics

#### Flashloans

Flashloans have been implemented in the `main` branch as a concept, but there is no set timeline for actually implementing them in the AMM protocol.

The implementation is based on Move's loan concept, where a Move object containing the loan data is issued but cannot be stored, copied, cloned or dopped, the only available action being to return the object back to the `pay_flashloan` function, which will verify the resulting constant product function.

The concept was explained early on by the Pontem team in this [Medium article](https://medium.com/p/bbc48a48d93c).

#### VE(3,3)

The `ve(3,3)` logic is currently under active development and can undergo significant changes; stay tuned.

#### Oracle

Liquidswap's oracle model is based on the cumulative price algorithm used by Uniswap v2. For this reason, token price variables on Liquidswap can overflow, just like on Uniswap.

The following fields in the Liquidity Pool resource are responsible for storing prices:

```
last_block_timestamp: u64,
last_price_x_cumulative: u128,
last_price_y_cumulative: u128,
```

Meanwhile, the Liquidity Pool module's function `get_cumulative_prices<X, Y, LP>` extracts prices from the pool.

If you are interested in the Oracle implementation, refer to our [integration](../integration/) guides.

#### Treasury

Source Code: [.sources/swap/dao\_storage.move](https://github.com/pontem-network/liquidswap/blob/main/sources/swap/dao\_storage.move)

The DAO Treasury receives a 0.1% fee from each swap transaction in every liquidity pool on the protocol. A reminder: Liquidswap's total fee is 0.3%, out of which 0.2% go to the liquidity providers and 0.1% to the treasury.

The treasury is currently managed by an admin multisig and will be eventually transfereed to a full-fledged Pontem DAO.

#### Emergency

Source Code: [.sources/swap/emergency.move](https://github.com/pontem-network/liquidswap/blob/main/sources/swap/emergency.move)

Can be used to pause/resume all swaps via an emergency account.

The contract itself can be disabled forever once the team can verifies that the protocol is fully stable and will not be derailed by any liquidity event, attack etc.
