# Smart Contracts Integration

* Smart contracts can be found in [Liquidswap](https://github.com/pontem-network/liquidswap) repository.
* The current version of Liquidswap deployed on address `0x43417434fd869edee76cca2a4d2301e528a1551b1d719b75c350c3c97d15b8b9.`
* Latest release tag can be found on [Releases](https://github.com/pontem-network/liquidswap/releases) page.

Before we start create new Move project so you can repeat the same steps.

### Add as dependency

To integrate Liquidswap into your project you first need to add dependency to your `Move.toml`

```
[dependencies.Liquidswap]
git = 'https://github.com/pontem-network/liquidswap.git'
rev = 'latest version'
```

Let's add also test coins we use in liquidswap (it's [liquidswap-lp](https://github.com/pontem-network/liquidswap-lp) repository):

<pre><code><strong>[dependencies.LiquidswapLP]
</strong>git = 'https://github.com/pontem-network/liquidswap-lp.git'
rev = 'latest version'</code></pre>

Test coins deployed already deployed on testnet and can be used in your project:

* `0x43417434fd869edee76cca2a4d2301e528a1551b1d719b75c350c3c97d15b8b9::coins::BTC`
* `0x43417434fd869edee76cca2a4d2301e528a1551b1d719b75c350c3c97d15b8b9::coins::USDT`

Now we can continue with the rest of things. Try to compile, you musn't see any errors.

_The liquidswap contracts always points to the latest published devnet revision, so if you have troubles after adding our dependencies remove `dependencies.AptosFramework` at all._

