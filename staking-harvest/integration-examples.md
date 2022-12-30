# Integration Examples

**Important:** you need to have a very good idea of what you are doing, because there are some risks.&#x20;

Use only test coins for test purposes. Your rewards or NFTs can get stuck in the staking pool if you use a wrong configuration, and you won't be able to withdraw them. First check that everything works on testnet and only then switch to the production environment! :warning:

We have prepared a few integration examples in case you want to integrate staking contracts into your Move smart contracts.

### Add as dependency

Add the Harvest package as a dependency to your project updating `Move.toml`:

```toml
[dependencies.Harvest]
git = 'https://github.com/pontem-network/harvest.git'
rev = 'latest version'
```

Replace `latest version` with the actual one - see the latest version on the [Releases](https://github.com/pontem-network/harvest/releases) page.

As we are going to use Liquidswap LP coins in the example (though it's not mandatory for harvest contracts), add another dependency:

```toml
[dependencies.LiquidswapLP]
git = 'https://github.com/pontem-network/liquidswap.git'
subdir = 'liquidswap_lp/'
rev = 'v0.4.3'
```

### Register pool / stake

**Important:** The following examples show how you can iterate with staking contracts in your smart contracts/product. The example is designed for educational purposes only; your code will probably be very different.&#x20;

Let's say we want to register a pool that uses Liquidswap LP coins as staking coins and reward coins as an unknown generic. After that, we will immediately stake.

{% embed url="https://gist.github.com/borispovod/84bb62d9160db6f0863a9f7604464d3f" %}
Register pool & stake
{% endembed %}

### Harvest

If we want to harvest the rewards, we can extend the module above with the harvest function:

{% embed url="https://gist.github.com/borispovod/53ba39f725f0ac968ee101e4a6a8c278" %}
Harvest
{% endembed %}

### Unstake

Once one week has elapsed from the moment of staking or once the established harvesting period is complete, you can unstake:

{% embed url="https://gist.github.com/borispovod/0f46c0a3164f5bb3a4ab5d71600b974d" %}
Unstake
{% endembed %}

### Boost

To boost or remove a boost, look at the following example where we extract boost collection from config:

{% embed url="https://gist.github.com/borispovod/9d312f1ec40638beeddd3bac4b52c0b5" %}

### More

You can find more examples in our [tests](https://github.com/pontem-network/harvest/tree/main/tests).
