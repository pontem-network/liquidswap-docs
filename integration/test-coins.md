# Test Coins

**This part works only for testnet, as on mainnet there is production ready coins should be used!**

During the integration, you probably would need test coins, so we prepared a repository containing several coins (USDT, BTC, USDC, DAI, ETH) just for tests.

The existing pools on Liquidswap use our test coins, so it's a good idea to follow standard test coins during integration. &#x20;

Repository - [test-coins](https://github.com/pontem-network/test-coins)

The following test coins are already deployed on the testnet/devnet and can be used in your project:

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

#### Faucet

Repository - [faucet](https://github.com/pontem-network/faucet)

Tests coins are available on Aptos testnet/devnet, to get free coins just do calls to faucet or visit [Liquidswap DApp](https://testnet.liquidswap.com).&#x20;
