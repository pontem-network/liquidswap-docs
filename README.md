# Introduction

![Liquidswap UI](assets/liquidswap.png)

The [Liquidswap](https://liquidswap.com) protocol is AMM (Automated Market Maker), created to allow a decentralized exchange of cryptocurrencies on Aptos blockchain. The protocol was built from a set of smart contracts developed by the Pontem Network team, written in Move language, and published on Aptos devnet.

Supported features and benefits:

* Allows both uncorrelated (Uniswap V2-like) and stable swaps
* DAO treasury is implemented, which receive a part of every swap transaction fee.
* Unlike Uniswap, Liquidswap allows the creation of many pools with even the same pairs but on different addresses. It's one of the nuances of developing the protocol on Move language and can be a nice feature together with others like dynamic fees in the distant future. Yet, still, we think that pools presented in UI will be the most popular now.
* Written in Move language: new smart-contract language designed with security in mind.
* Large TPS because of Aptos parallel transaction execution mechanism.
* Partially covered with formal verification; the complete formal verification coverage is coming in the future.

## Devnet version

The current version of Liquidswap was deployed on Aptos devnet and published on [https://liquidswap.com](https://liquidswap.com).

Visit it and try it yourself. Provided feedback would be appreciated 😊

## Developer Links

To be more familiar with Liquidswap on the top level, start with [Protocol Overview](protocol-overview.md).

To be more to the point, look at our tutorials:

* [Smart contracts usage & integrations](integration/)
* [Frontend integration](typescript-sdk.md)

If you are looking for source code, visit our [Github](https://github.com/pontem-network):

* [Liquidswap Smart Contracts](https://github.com/pontem-network/liquidswap)
* [Liquidswap LP & tests coin](https://github.com/pontem-network/liquidswap-lp)
* [Typescript SDK](https://github.com/pontem-network/liquidswap-sdk)

If you need help integrating your service with Liquidswap, feel free to contact us:

* [Telegram](https://t.me/pontemnetworkchat)
* [Discord](https://discord.gg/44QgPFHYqs)
