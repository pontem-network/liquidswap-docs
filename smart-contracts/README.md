# Smart Contracts

The contracts are published in the Liquidswap GitHub repository at [https://github.com/pontem-network/liquidswap](https://github.com/pontem-network/liquidswap).

Liquidswap uses smart contracts written in the Move language and executed by Move VM. The core contracts containing the protocol's logic and safety guarantees and are separated from the periphery contracts, allowing for a well-tested, reasonably small core layer and multiple versions of the periphery that can work with  the core utilizing their own custom logic.

For example, the basic `Router.move`, used by Liquidswap's frontend, proxys all Liquidity Pool module methods with additional safety and price checks. At the same time, traders who want to minimize gas usage can write their own custom routers or call the core contracts directly.

## Branches and versions

The current `main` branch is the development branch and always contains the latest changes.

Release branches contain the latest changes created for the specific release.

The current release tags `v0.4*` are designed for the mainnet release.

You can always find the latest release in the [Releases](https://github.com/pontem-network/liquidswap/releases) section.

### Addresses

All liquidity pools resources and LP coins currently placed at the following resource account:

```
0x05a97986a9d031c4567e15b797be516910cfcb4156312482efc6a19c0a30c948
```

Liquidswap modules are deployed at the following address:

```
0x190d44266241744264b964a37b8f09863167a12d3e70cda39376cfb4e3561e12
```

**All smart contracts and dependencies (exclude that placed on `0x1` address) are immutable.**

## Integrations

This document explains how Liquidswap contracts are organized at the top level.

If you are already familiar with this material, you can proceed to the [integrations](../integration/) docs.

## Liquidity Pool

Source Code: [./sources/swap/liquidity\_pool.move](https://github.com/pontem-network/liquidswap/blob/main/sources/swap/liquidity\_pool.move).

The Liquidity Pool module implements all swap logic, LP logic, math (constant product formula), and core checks.

The core part of the Liquidity Pool contract is a resource that describes the liquidity pools themselves and contains all the required information:

{% code lineNumbers="true" %}
```rust
struct LiquidityPool<phantom X, phantom Y, phantom Curve> has key {
    coin_x_reserve: Coin<X>,
    coin_y_reserve: Coin<Y>,
    last_block_timestamp: u64,
    last_price_x_cumulative: u128,
    last_price_y_cumulative: u128,
    lp_mint_cap: coin::MintCapability<LP<X, Y, Curve>>,
    lp_burn_cap: coin::BurnCapability<LP<X, Y, Curve>>,
    // Scales are pow(10, token_decimals).
    x_scale: u64,
    y_scale: u64,
    locked: bool,
    fee: u64,           // 1 - 100 (0.01% - 1%)
    dao_fee: u64,       // 0 - 100 (0% - 100%)
}
```
{% endcode %}

* Reserves of both tokens `X` and `Y`.
* Curve type: `phantom Curve`.
* Cumulative price information.
* Mint and burn capabilities for `LP` coins.
* Token decimal scales (used for stable swaps).
* If a pool is locked (needed for the flashloans).

New liquidity pools are created on the registrar account, and the corresponding resource is placed in that account's storage.

Almost like Uniswap the Liquidswap creates pools on the reserved address:

```
0x05a97986a9d031c4567e15b797be516910cfcb4156312482efc6a19c0a30c948
```

All pools are unique, so there can't be two pools containing the same coins, and it also works for LP and LP generics.

To avoid confusion, Liquidswap uses a frontend interface with a list of pools already pre-filled and a pools registry to show the users mostly verified pools.

### Generics

A set of `(X, Y, Curve)` uniquely identifies a liquidity pool on the blockchain.

Because of that, all functions that refer to pools have three generic parameters:

```
public entry fun swap<X, Y, Curve>(...) {} 
```

* `X, Y` - the tokens to be swapped in a pool, for example, `0x1::aptos_coin::AptosCoin` and `test_coins::coins::USDT`(wrapped USDT)
* `Curve` - type of the Curve corresponding to the pool, explained below.
* Note: **Generics X and Y must be sorted**, see [Coin Sorting](./#coins-sorting).

### Functions

**In most cases, you won't need to use the `LiquidityPool` module itself. The `Router` module provides high-level wrappers around `LiquidityPool`, which simplify most of the tasks.**

Operations with liquidity:

* `register<X, Y, Curve>` - creates fora new liquidity pool on reserved account.
* `mint<X, Y, Curve>` - mints new LP coins in exchange for `Coin<X>` and `Coin<Y>`.
* `burn<X, Y, Curve>` - burns some LP coins in exchange for `Coin<X>` and `Coin<Y>`.
* `swap<X, Y, Curve>` - swaps `Coin<X>` or `Coin<Y>` or both to get `Coin<X>` or `Coin<Y>` or both in exchange.
* `flashloan<X, Y, Curve>` - flashloan reserves `Coin<X>` and/or `Coin<Y>` from the pool, also returns `Flashloan` object.&#x20;
* `pay_flashloan<X, Y, Curve>` - pay flashloan by returning `Coin<X>`, `Coin<Y>` and `Flashloan` object.

Getters:

* `is_pool_locked<X, Y, Curve>` - returns bool determining if the pool is locked (flashloaned).
* `get_reserves_size<X, Y, Curve>` - returns the current reserves size for both `Coin<X>` and `Coin<Y>`.
* `get_curve_type<X, Y, Curve>` - returns the curve type for the pool.
* `get_decimals_scales<X, Y, Curve>` - returns the decimal scales for both `Coin<X>` and `Coin<Y> - but` correct values are returned only for stable pools.
* `pool_exists_at<X, Y, Curve>` - returns bool determining if the pool exists.
* `get_cumulative_prices<X, Y, Curve>` - return cumulative prices and the last block's timestamp.
* `get_fees_config<X, Y, Curve>` - returns fees config for the pool (both nominator fee and denominator).
* `get_fee<X, Y, Curve>` - return the fee nominator for pool.
* `get_dao_fees_config<X, Y, Curve>` - returns dao fees config for the pool (both nominator fee and denominator).
* `get_dao_fee<X, Y, Curve>` - return the dao fee nominator for pool.&#x20;

#### Coin Sorting

To work with Liquidity Pool module functions, you must sort the coins you pass on as generics.

Sorting is important as it brings in the rules for creating liquidity pools. All functions in the Liquidity Pool module accept only sorted generics; otherwise, it reverts.

For the Router and Scripts modules, **only part** of the functions requires generics to be sorted.

The current sorting algorithm takes the types of provided coins, e.g., `address::module::struct_name`, and compares the struct name of both coins. If it's equal, it continues with the module name and, in the end, with the address.

You always can just use the implementation in Coin Helper module: [coin\_helper.move](https://github.com/pontem-network/liquidswap/blob/main/sources/libs/coin\_helper.move#L44).

Other languages:

* Implementation in Javascript: [Coin Sorting In JS](https://gist.github.com/borispovod/2809728c8959649d42c5cef15b4cedb7).
* Also, it's supported in [Typescript SDK](../typescript-sdk.md).

### Curve types

Source code: [sources/swap/curves.move](https://github.com/pontem-network/liquidswap/blob/main/sources/swap/curves.move)

When creating a pool, the curve type can be provided; it's as simple as using a generic to determine the pool type.

The types themself:

```rust
/// For pairs like BTC, Aptos, ETH.
struct Uncorrelated {}

/// For stablecoins like USDC, USDT.
struct Stable {}
```

* `liquidswap::curves::Uncorrelated`- Uncorrelated curve type.
* `liquidswap::curves::Curve` - Stable curve type.

The following types can be imported into code and should be used as the last generic argument in almost all modules functions.

Functions:

* `is_uncorrelated<Curve>` - returns `true` if provided type is `Uncorrelated`.
* `is_stable<Curve>` - returns true if provided type is `Stable`.
* `is_valid_curve<Curve>` - returns true if provided type is `Stable` or `Uncorrelated`.
* `assert_valid_curve<Curve>` - abort if provided type is not a curve type.

You can read more about Liquidswap's curve formulas on the [Protocol Overview](../protocol-overview.md) page.

### LP coins

Source code: - [liquidswap\_lp/sources/lp\_coin.move](https://github.com/pontem-network/liquidswap/blob/main/liquidswap\_lp/sources/lp\_coin.move)

_Consider that LiquidswapLP is an independent module inside the liquiswap repository, so it must be imported as a separate dependency._

The `LP` coins type for pairs `X` and `Y` is represented as `LP<X, Y, Curve>` and deployed on the following resource account:

```
0x385068db10693e06512ed54b1e6e8f1fb9945bb7a78c28a45585939ce953f99e
```

The LP coin registers automatically for each new liquidity pool, and generics are always sorted.

For `APT/BTC` uncorrelated pool, the LP coin would look so:

```

0x385068db10693e06512ed54b1e6e8f1fb9945bb7a78c28a45585939ce953f99e::lp_coin::LP<
    0x43417434fd869edee76cca2a4d2301e528a1551b1d719b75c350c3c97d15b8b9::coins::BTC,
    0x1::aptos_coin::AptosCoin,
    0x43417434fd869edee76cca2a4d2301e528a1551b1d719b75c350c3c97d15b8b9::curves::Uncorrelated,
>
```

### Flashloans

The implementation is based on Move's loan concept, where a Move object containing the loan data is issued but cannot be stored, copied, cloned or dropped, the only available action being to return the object back to the `pay_flashloan` function, which will verify the resulting constant product function.

The concept was explained early on by the Pontem team in this [Medium article](https://medium.com/p/bbc48a48d93c).

Read more in the [integration section](../integration/flashloans.md).

## Router

Source code: [./sources/swap/router.move](https://github.com/pontem-network/liquidswap/blob/main/sources/swap/router.move).

The Router module is a periphery layer on top of the Liquidity Pool module.

The Router contains additional checks to verify that the amount that a developer, trader, or liquidity provider wants to exchange/mint/burn is reasonable.

Also, the module sorts tokens or coins automatically for **part of functions** and has several useful getters that help estimating the swap price.

In most cases, we recommend using a Router if you want to work with the Liquidity Pool module.

Third-party teams can provide their own routers, but Liquidswap's standard Router should be enough for most cases. An added advantage is that it's already audited.

### Functions

The router functions accept the resources `Coin<X>,` `Coin<Y>`, `Curve` similar to Liquidity Pool module.&#x20;

However, the functions cannot be called directly from a transaction; if you need entry points, refer to [Scripts](./#scripts).

Developers interacting with **swap functions and getters** can order generics in any way they wish. For example, if one wants to swap `aptos_framework::aptos_coin::AptosCoin` to `test_coins::coins::USDT, one`can just use the function:

```
router::swap_exact_coin_for_coin<
    aptos_framework::aptos_coin::AptosCoin,
    test_coins::coins::USDT,
    curves::Uncorrelated,
>
```

A reverse swap (`USDT` -> `APTOS`) can be done by reordering the generics:

```
router::swap_exact_coin_for_coin<
    test_coins::coins::USDT,
    aptos_framework::aptos_coin::AptosCoin,
    curves::Uncorrelated,
>
```

**Important note:** the rest of functions (`register_pool`,`add_liquidity`, `remove_liquidity`) requires **sorted generics!**

Liquidity operations:

* **Requires sorted generics:**
  * `register_pool<X, Y, Curve>` - register a new pool with `Coin<X>` and `Coin<Y>` as reserves and provided curve.
  * `add_liquidity<X, Y, Curve>` - add liquidity (`Coin<X>` and `Coin<Y>`) to an existing pool.
  * `remove_liquidity<X, Y, Curve>` - burn `Coin<LP<X ,Y, Curve>>` and get `Coin<X>` and `Coin<Y>` back with checks.
* **Generics can be sorted in any way:**&#x20;
  * `swap_exact_coin_for_coin` - swap an exact amount of `Coin<X>` to get no less than the specified amount of `Coin<Y>`.
  * `swap_coin_for_exact_coin` - swap no more than the specified maximum amount of `Coin<X>` to get an exact amount of `Coin<Y>`.
  * `swap_coin_for_coin_unchecked` - simply swaps tokens without any checks.

**Getters:**

* `get_amount_out<X, Y, Curve>` - estimate the amount of `Y` tokens resulting from a swap of a specified amount in`X` tokens. Can consume a lot of gas when used for stable pools.
* `get_amount_in<X, Y, Curve>` - estimate the amount of `X` tokens needed to get a specified amount in`Y` tokens.
* `calc_optimal_coin_values<X, Y, Curve>` - calculates the optimal amounts of `Coin<X>` and `Coin<Y>` needed to add liquidity and get a fair amount of LP coins.
* `get_reserves_size<X, Y, Curve>` - returns the reserves for both coin X and coin Y held by the pool.
* `pool_exists_at<X, Y, Curve>` - returns bool determining if the pool exists.
* `get_decimals_scales<X, Y, Curve>` - returns the pool's decimals scale. The resulting values will be correct only for stable pools.
* `get_cumulative_prices<X, Y, Curve>` - returns cumulative prices and the last block's timestamp.
* `get_reserves_for_lp_coins<X, Y, Curve>` - returns the amounts in `Coin<X>` and `Coin<Y>` that a user will receive after burning LP coins.
* `get_fees_config<X, Y, Curve>` - returns fees config for the pool (both nominator fee and denominator).
* `get_fee<X, Y, Curve>` - return the fee nominator for pool.
* `get_dao_fees_config<X, Y, Curve>` - returns dao fees config for the pool (both nominator fee and denominator).
* `get_dao_fee<X, Y, Curve>` - return the dao fee nominator for pool.

## Scripts

Source code: [./sources/swap/scripts.move](https://github.com/pontem-network/liquidswap/blob/main/sources/swap/scripts.move).

The top-level module contains the entry functions that users can execute directly by sending transactions and that third-party modules can use via `&signer`.

Summary:

* Requires a signer;
* Accepts numbers as arguments representing coins' values;
* Extracts coins/tokens directly from the signer account;
* Registers a LP token on the account if it's not registered;
* Works with default [Router](./#router).

This is the optimal way to interact with Liquidswap if you want to call a swap from the CLI or the UI using the standard router.

### Functions

All the functions have 'slippage' amount arguments, like `coin_x_val_min`, `coin_y_val_min` or `min_x_out_val`, etc.

* **Requires sorted generics:**
  * `register_pool<X, Y, Curve>` - register a pool with the specified curve type.
  * `register_pool_and_add_liquidity<X, Y, Curve>` - register a pool with the specified curve type and add liquidity immediately (extracted from the signer's balance), with the resulting LP tokens deposited to the signer.
  * `add_liquidity<X, Y, LP>` - add liquidity to an existing pool, with the resulting LP coins deposited to the signer's address.
  * `remove_liquidity<X, Y, Curve>` - burns users LP coins and deposit received coin X and coin Y on user account.
* **Generics can be sorted in any way:**&#x20;
  * `swap<X, Y, Curve>` - swap `Coin<X>` for `Coin<Y>;` the developer has to provide the amount of `X` coins `coin_val` to get the minimum amount `coin_out_min_val` of Y.
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
* `generate_lp_name<X, Y, Curve>` - generates a name and symbol for the `LP` token from `X` and `Y` symbols and `Curve`.

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

The smart contracts utilize the following math libraries created by the Pontem team:

* [u256](https://github.com/pontem-network/u256) - a pure Move `u256` implementation because some numbers don't fit in `u128`.
* [uq64x64](https://github.com/pontem-network/uq64x64) - the Q number format for cumulative price calculations.

### Advanced topics

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

Meanwhile, the Liquidity Pool module's function `get_cumulative_prices<X, Y, Curve>` extracts prices from the pool.

If you are interested in the Oracle implementation, refer to our [integration](../integration/) guides.

#### Treasury

Source Code: [.sources/swap/dao\_storage.move](https://github.com/pontem-network/liquidswap/blob/main/sources/swap/dao\_storage.move)

The DAO Treasury receives a fee from each swap transaction in every liquidity pool on the protocol.

The treasury is currently managed by Treasury multisig and will be eventually transfered to a full-fledged Pontem DAO.

#### Config & Dynamic Fees

Source code: [./sources/swap/global\_config.move](https://github.com/pontem-network/liquidswap/blob/main/sources/swap/global\_config.move)

Most of the functions mostly needed to migrate Treasury accounts from one to another and change default fees.&#x20;

The [liquidity pool](https://github.com/pontem-network/liquidswap/blob/main/sources/swap/liquidity\_pool.move) module contains functions to change individual pools fees.

Read more fees concept: [Fees & Treasury](../protocol-overview.md#fees-and-treasury).

#### Emergency

Source Code: [.sources/swap/emergency.move](https://github.com/pontem-network/liquidswap/blob/main/sources/swap/emergency.move)

Can be used to pause/resume all swaps via an emergency account.

The contract itself can be disabled forever once the team can verifies that the protocol is fully stable and will not be derailed by any liquidity event, attack etc.
