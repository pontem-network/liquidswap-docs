# Liquidswap DApp

<figure><img src=".gitbook/assets/Снимок экрана 2022-09-06 в 18.50.14.png" alt=""><figcaption><p>Liquidswap UI/UX</p></figcaption></figure>

The user interface for interacting with the Liquidswap protocol is published at [https://liquidswap.com/](https://liquidswap.com/)

It supports most of Liquidswap's core features:

* Pool creation
* Adding liquidity
* Burning liquidity
* Swaps
* Basic stats

**The next sections are not actual for v0.3.0, and we are currently rewriting them.**&#x20;

#### Coin Registry

The coin registry currently used by Liquidswap is the [Pontem Coin Registry](https://github.com/pontem-network/coins-registry).

It contains not only coins and tokens themselves but also a list of the supported pools.

Feel free to make a [Pull Request](https://github.com/pontem-network/coins-registry/pulls) with your own pool or token.

#### Adding Custom Pools and Tokens

To connect the Liquidswap interface to a Custom Pool or a custom coin registry:

1. Click on 'Select Token'.
2. Click on the 'Manage Pools' button at the end of the modal.
3. Click on 'Add Pool.
4. Fill in the pool's details manually or upload a list of assets.

**How to add a pool you've just created**

1. `Token 1`: the path to token `X` in your pool, e.g.: `0x1::aptos_coin::APTOS`
2. `Token 2`: the path to token `Y` in your pool, e.g.: `0x43417434fd869edee76cca2a4d2301e528a1551b1d719b75c350c3c97d15b8b9::coins::USDT`
3. `LP Token`: the path to the LP token - usually it's `address::lp::LP<path to coin X, path to coin Y>`.
