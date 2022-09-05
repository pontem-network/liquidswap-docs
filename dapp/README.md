# Liquidswap DApp

<figure><img src="../.gitbook/assets/Снимок экрана 2022-09-05 в 09.05.46.png" alt=""><figcaption><p>Liquidswap DApp UI</p></figcaption></figure>

The user interface to interact with the Liquidswap protocol is published on [https://liquidswap.com/](https://liquidswap.com/)

It supports most of the Liquidswap core features:

* Creation of new pools.
* Add liquidity.
* Burn of liquidity.
* Swaps.
* Some basic statistic.

### Coins Registry

The coins registry currently used by Liquidswap it's [Pontem Coin Registry](https://github.com/pontem-network/coins-registry).&#x20;

It contains not only coins themselves but also a list of supported pools.

Feel free to make a [Pull Request](https://github.com/pontem-network/coins-registry/pulls) with your pool or coin.

### Adding Custom Pool and Tokens

To connect Liquidswap interface with Custom Pool or custom coins registry:

1. Click on "Select Token".
2. Click on the "Manage Pools" button at the end of the modal.
3. Click on "Add Pool".
4. Fill pool details manually or by uploading a coins list.

<figure><img src="../.gitbook/assets/Снимок экрана 2022-09-05 в 09.18.10.png" alt=""><figcaption><p>Adding new pool to Liquidswap</p></figcaption></figure>

#### How to add a pool you just created

1. `Pool Creator Address` - It's your address.
2. `Token 1` - It's path to coin `X` of your pool, e.g.: `0x1::aptos_coin::APTOS`
3. `Token 2` - It's path coin `Y` of your pool, e.g.: `0x43417434fd869edee76cca2a4d2301e528a1551b1d719b75c350c3c97d15b8b9::coins::USDT`
4. `LP Token` - it's path to LP coin, but usually it's: `address::lp::LP<path to coin X, path to coin Y>`.

