# Smart Contracts Integration

* Smart contracts source code can be found in [Liquidswap](https://github.com/pontem-network/liquidswap) repository.
* The current version of Liquidswap deployed on address `0x43417434fd869edee76cca2a4d2301e528a1551b1d719b75c350c3c97d15b8b9.`
* The latest release tag can be found on [the Releases](https://github.com/pontem-network/liquidswap/releases) page.

Before we start, create a new Move project so you can repeat the same steps.

### Add as dependency

To integrate Liquidswap into your project, you first need to add a dependency to your `Move.toml`

{% code lineNumbers="true" %}
```toml
[dependencies.Liquidswap]
git = 'https://github.com/pontem-network/liquidswap.git'
rev = 'latest version'
```
{% endcode %}

Also, let's add test coins we use in liquidswap (it's [liquidswap-lp](https://github.com/pontem-network/liquidswap-lp) repository):

{% code lineNumbers="true" %}
```toml
[dependencies.LiquidswapLP]
git = 'https://github.com/pontem-network/liquidswap-lp.git'
rev = 'latest version'
```
{% endcode %}

Replace `'latest version'` with actual ones from repositories.

Test coins deployed already deployed on testnet and can be used in your project:

```
// Test BTC
0x43417434fd869edee76cca2a4d2301e528a1551b1d719b75c350c3c97d15b8b9::coins::BTC

// Test USDT
0x43417434fd869edee76cca2a4d2301e528a1551b1d719b75c350c3c97d15b8b9::coins::USDT
```

Now we can continue with the rest of things. Try to compile. You mustn't see any errors.

_The Liquidswap contracts always point to the latest published devnet revision, so if you have troubles after adding our dependencies, remove `dependencies.AptosFramework` at all._

