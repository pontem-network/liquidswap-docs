# Integration Examples

**Important:** you must clearly understand what you are doing, as there are many risks; use only test coins for test purposes, as your rewards or NFTs can get stuck in the staking pool because of the wrong configuration, and you wouldn't be able to withdraw them. First test and only then go to production environment! :warning:

We prepared integration examples if you want to integrate staking contracts into your Move smart contracts.

### Add as dependency

Add Harvest package as a dependency to your project updating `Move.toml`:

```toml
[dependencies.Harvest]
git = 'https://github.com/pontem-network/harvest.git'
rev = 'latest version'
```

Replace `latest version` with real one - see the latest version on [Releases](https://github.com/pontem-network/harvest/releases) page.

As we are going to use Liquidswap LP coins in the example (it's not mandatory for harvest contracts), add another dependency:

```toml
[dependencies.LiquidswapLP]
git = 'https://github.com/pontem-network/liquidswap.git'
subdir = 'liquidswap_lp/'
rev = 'v0.4.3'
```

### Register pool / Stake

**Important:** The following examples show how you can interact with staking contracts in your smart contracts/product. The example done for educational purposes, your code would be very different.&#x20;

Let's say we want to register a pool that will use Liquidswap LP coins as staking coins and reward coins as unknown generic. After that, we will immediately stake.

{% embed url="https://gist.github.com/borispovod/84bb62d9160db6f0863a9f7604464d3f" %}
Register pool & stake
{% endembed %}

### Harvest

If we want to harvest, we can extend the module above with the harvest function:

{% embed url="https://gist.github.com/borispovod/53ba39f725f0ac968ee101e4a6a8c278" %}
Harvest
{% endembed %}

### Unstake

After one week or harvest period finishing you can do the unstake:

{% embed url="https://gist.github.com/borispovod/0f46c0a3164f5bb3a4ab5d71600b974d" %}
Unstake
{% endembed %}

### Boost

To boost and remove boost, look at the following example where we extract boost collection from config:

{% embed url="https://gist.github.com/borispovod/9d312f1ec40638beeddd3bac4b52c0b5" %}

### More

You can find more examples in our [tests](https://github.com/pontem-network/harvest/tree/main/tests).
