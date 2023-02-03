# Coins Registry

We are developing our own [Coin Registry](https://github.com/pontem-network/coins-registry) file formats for [LiquidSwap](https://liquidswap.com/), which contains all the information about tokens and pools in which currencies can be exchanged.

Currently, we use two files for data:

* [coins.json](https://github.com/pontem-network/coins-registry/blob/main/src/coins.json)
* [pools.json](https://github.com/pontem-network/coins-registry/blob/main/src/pools.json)

And an [index.js](https://github.com/pontem-network/coins-registry/blob/main/src/index.js) file for easier usage of coins-registry in other projects. Through the interface, it is easy to get tokens or pools lists for the selected chain type. It provides two methods: `getCoinsFor` and `getPoolsFor`. Each of these methods requires the network type as an argument. Possible values for network types: `mainnet` and `testnet`.

### Coins Data Structure

There is a coins list in coins.json. One coin can be presented as follows:

```
{
    "source": "aptos",
    "name": "Aptos Coin",
    "chainId": 1,
    "decimals": 8,
    "symbol": "APT",
    "type": "0x1::aptos_coin::AptosCoin",
    "caution": false,
    "order": 1
}
```

#### Fields Description

`source` - enum data type with strict value check. If you’d like to add the new value into it, fill the form. Usually source is the name of a partner's company to add many coins into the list. We support now following sources:

* `aptos` - for the native Aptos Coin.
* `partners` - for other tokens.
* `celer` - for tokens provided by Celer.
* `layerzero` - for tokens provided by Layer Zero.
* `wormhole` - for tokens provided by Wormhole.

`name` - tokens' full name. We update this value by a request to the node. It is used on Picture 1 as a bottom string after the dot (on the picture it is Bitcoin).

`chainId` - for mainnet value is 1. Currently, testnet tokens stored in the `testnet` branch of this repo.

`decimals` - decimals how supported by your token. We update this value by a request to the node.

`symbol` - it is used to construct a token alias, which can be equal to symbol as on Picture 1 in the top string.

`type` - full type of token. String with following structure ADDRESS::MODULE::COIN, e.g. `0x1000000fa32d122c18a6a31c009ce5e71674f22d06a581bb0a15575e6addadcc::usda::USDA`.

`caution` - if we need to show warning icon near the token - we will add the caution field.

`order` - order of a token in the tokens list. Current logic:

```
1 Apt
10 USDC
20 USDT
30 DAI
40 BTC
50 WETH
60 BUSD / BNB
1000 other tokens
```

### Pools Data Structure

```
{
    "coinX": "0xf22bede237a07e121b56d91a491eb7bcdfd1f5907926a9e58338f964a01b17fa::asset::USDT",
    "coinY": "0x1::aptos_coin::AptosCoin",
    "curve": "uncorrelated",
    "networkId": 1
}
```

#### Fields Description

`coinX` - a full type of coin. String with the following structure ADDRESS::MODULE::COIN, e.g. `0xf22bede237a07e121b56d91a491eb7bcdfd1f5907926a9e58338f964a01b17fa::asset::USDT`.

`coinY` - a full type of coin. String with the following structure ADDRESS::MODULE::COIN, e.g. `0x1::aptos_coin::AptosCoin`.

> ⚠️ Coins should be sorted.

`curve` - to strictly point curve type use the following values:

* selectable - will keep a switcher for a curve type available for declared coins pair.
* uncorrelated - keep only an uncorrelated curve pool on UI.
* stable - keep only a stable curve pool on UI.
