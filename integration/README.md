# Integration

* The smart contracts' source code can be found in the [Liquidswap](https://github.com/pontem-network/liquidswap) repository.
*   The current version of Liquidswap is deployed on the address&#x20;

    ```
    0x190d44266241744264b964a37b8f09863167a12d3e70cda39376cfb4e3561e12
    ```
* The latest release tag can be found on the[ Releases](https://github.com/pontem-network/liquidswap/releases) page.

Before we start, create a new Move project so that you can repeat the steps as described here.

#### Supported networks

The Aptos mainnet and testnet, devnet are currently supported.&#x20;

For both testnet and mainnet you can use latest tags, for [devnet](../smart-contracts/#devnet) see details.

### Add as dependency

To integrate Liquidswap into your project, you first need to add a dependency to your `Move.toml`

{% code lineNumbers="true" %}
```toml
[dependencies.Liquidswap]
git = 'https://github.com/pontem-network/liquidswap.git'
rev = 'latest version'
```
{% endcode %}

Import [Router v2:](../smart-contracts/#router-v2)

```rust
[dependencies.LiquidswapRouterV2]
git = 'https://github.com/pontem-network/liquidswap.git'
subdir = 'liquidswap_router_v2/'
rev = 'latest version'
```

Add the LP coin module as a dependency too:

```toml
[dependencies.LiquidswapLP]
git = 'https://github.com/pontem-network/liquidswap.git'
subdir = 'liquidswap_lp/'
rev = 'latest version'
```

**Replace `'latest version'` with the actual ones from the repositories.**

Next, try compile; you shouldn't get any errors.

_Liquidswap's contracts always point to the latest `AptosFramework` devnet revision. If you encounter issues after adding our dependencies, remove `dependencies.AptosFramework` completely from your project._

