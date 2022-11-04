# Introduction

![Liquidswap UI](assets/liquidswap.png)

[Liquidswap](https://liquidswap.com) is the first AMM (Automated Market Maker) on the Aptos blockchain, created to enable safe and decentralized token swaps. The protocol uses smart contracts developed by the Pontem Network team, written in the Move language, and published on the Aptos mainnet.

### Features

Supported features and benefits:

* Uncorrelated token swaps (similar to Uniswap V2);
* Stable swaps for correlated assets, using a different liquidity curve;
* A DAO treasury that receives a part of every swap transaction fee;
* Dynamic fees: managed by treasury multisig and dao in the future.
* Written in Move: a new smart contract language designed with security in mind;
* High speed thanks to Aptos' parallel transaction execution engine;
* Formal verification partially complete: full formal verification is coming in the future.

## Mainnet :tada:



The current version of Liquidswap is deployed on the Aptos mainnet and located at [https://liquidswap.com](https://liquidswap.com).

The mainnet has been available since 19.10.2022; the [contracts](https://github.com/pontem-network/liquidswap) since v0.4.2 deployed to mainnet.

Please test Liquidswap for yourself and share your feedback - it will be greatly appreciated 😊

## Security audits

We've done 2 security audits and another one in progress currently.

### Ottersec

Website - [OtterSec.io](https://osec.io/)

Audit report:

{% file src=".gitbook/assets/pontem-liquidswap-ottersec-audit (1).pdf" %}

The rest of the changes, like flashloans and dynamic fees and configuration, are also reviewed by the Ottersec team but not added to the report.

### Halborn

Website - **** [Halborn.com](https://halborn.com/)

Audit reports:

**General**

{% file src=".gitbook/assets/pontem-liquidswap-halborn-audit.pdf" %}

**Flashloan**

{% file src=".gitbook/assets/pontem-liquidswap-flashloan-halborn-audit.pdf" %}

**Dynamic Fees and Config**

{% file src=".gitbook/assets/pontem-liquidswap-dynamic-fees-config.pdf" %}



### Zellic

Website - [https://www.zellic.io/](https://www.zellic.io/)

{% file src=".gitbook/assets/pontem-liquidswap-report-zellic.pdf" %}

## Developer Links

For a general idea about Liquidswap, start with the [Protocol Overview](protocol-overview.md).

To get hands-on with the DEX, check out the tutorials:

* [Smart contracts usage & integrations](integration/)
* [Frontend integration](typescript-sdk.md)

Looking for source code? Visit our [Github](https://github.com/pontem-network):

* [Liquidswap Smart Contracts](https://github.com/pontem-network/liquidswap)
* [Test coins](https://github.com/pontem-network/test-coins)&#x20;
* [Typescript SDK](https://github.com/pontem-network/liquidswap-sdk)

Need help integrating your service with Liquidswap? Feel free to contact us:

* [Telegram](https://t.me/pontemnetworkchat)
* [Discord](https://discord.gg/44QgPFHYqs)
