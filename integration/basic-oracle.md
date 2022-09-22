# Basic Oracle

As Liquidswap accumulates cumulative prices for each pair at the start of each block, in case liquidity operation happens on the pool, developers can build their oracle with it.

First of all, add [overflowing math](https://github.com/pontem-network/overflowing-math) as dependency to your project:

```toml
[dependencies.OverflowingMath]
git = "https://github.com/pontem-network/overflowing-math.git"
rev = "v0.1.0"
```

It's important to use overflowing math when works with cumulative prices of Liquidswap, as cumulative prices can overflow and it's ok.&#x20;

The basic example of oracle would be the following contract:

{% embed url="https://gist.github.com/borispovod/a11b1a9019ef26d7f23c7297a68c0066" %}
