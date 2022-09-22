# Integration

* The smart contracts' source code can be found in the [Liquidswap](https://github.com/pontem-network/liquidswap) repository.
* The current version of Liquidswap is deployed on the address `0x43417434fd869edee76cca2a4d2301e528a1551b1d719b75c350c3c97d15b8b9.`
* The latest release tag can be found on the[ Releases](https://github.com/pontem-network/liquidswap/releases) page.

Before we start, create a new Move project so that you can repeat the steps as described here.

### Add as dependency

To integrate Liquidswap into your project, you first need to add a dependency to your `Move.toml`

{% code lineNumbers="true" %}
```toml
[dependencies.Liquidswap]
git = 'https://github.com/pontem-network/liquidswap.git'
rev = 'latest version'
```
{% endcode %}

Add the LP coin module as a dependency too:

```toml
[dependencies.LiquidswapLP]
git = 'https://github.com/pontem-network/liquidswap.git'
subdir = 'liquidswap_lp/'
rev = 'release-v0.3'
```

**Replace `'latest version'` with the actual ones from the repositories.**

Next, try compile; you shouldn't get any errors.

#### **Test Coins**

Repository - [test-coins](https://github.com/pontem-network/test-coins)

The following test coins are already deployed on the devnet and can be used in your project:

```
// Test BTC
0x43417434fd869edee76cca2a4d2301e528a1551b1d719b75c350c3c97d15b8b9::coins::BTC

// Test USDT
0x43417434fd869edee76cca2a4d2301e528a1551b1d719b75c350c3c97d15b8b9::coins::USDT
```

Add as dependency:&#x20;

```toml
[dependencies.TestCoins]
git = 'https://github.com/pontem-network/test-coins.git'
rev = 'latest version'
```

Use in your code:

```rust
use test_coins::coins::{USDT, BTC};
use test_coins_extended::extended_coins::{USDC, ETH, DAI};
...
```

_Liquidswap's contracts always point to the latest `AptosFramework` devnet revision. If you encounter issues after adding our dependencies, remove `dependencies.AptosFramework` completely from your project._

