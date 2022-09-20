# Create Pool

You must deploy an LP coin module on your account to create a liquidity pool.

We suggest that you copy our [LP Template ](https://github.com/pontem-network/liquidswap-lp/blob/main/sources/lp.move)and simply deploy it under your account.

{% embed url="https://gist.github.com/borispovod/ccda44afd615cf5aeea17b4ed2458414" %}
LP Template
{% endembed %}

_Don't forget to replace `liquidswap_lp` in the module declaration with the correct address after deployment._

Now you can call on Router to create a new `APTOS`/`BTC` liquidity pool on your account.

Extend the example from the previous guide and add a new function:

{% embed url="https://gist.github.com/borispovod/5caea21cc3777372ce18cbb03cd6cac0" %}
Create liquidity pool
{% endembed %}

#### Creating a stable pool

In the same way, you can create a stable pool but use `1` as the `curve_type` argument.

{% embed url="https://gist.github.com/borispovod/a842a47f10a093d2591326b6ea85550c" %}
Create stable liquidity pool
{% endembed %}

### Add liquidity to your pool

Let's add liquidity to **our** new pool:

{% embed url="https://gist.github.com/borispovod/1fbb2f7c3fd76a50371241ea89700c64" %}
Add liquidity
{% endembed %}

As you can see above, `add_liquidity` allows to add liquidity to the newly created pool and then deposit the remainders of both `X` and `Y` tokens/coins back in the account, together with the new `LP` tokens.

_When working with real-world examples, it's better to deposit liquidity immediately during the same transaction with which you create a pool. We also recommend using slippage values for the amount of the LP tokens you'll get in exchange for your liquidity._

### Adding liquidity to an existing pool

Let's say we want to add liquidity to an existing `BTC/Aptos` pool created by the Pontem team.

The only difference is that we'll use the `@liquidswap` address as the pool address and a different LP.

{% embed url="https://gist.github.com/borispovod/c6abe785342fd3a18059d7e155647dd4" %}
Add liquidity to Pontem APTOS / BTC pool
{% endembed %}

_When working with real-world examples, it's better to deposit liquidity immediately during the same transaction with which you create a pool. We also recommend using slippage values for the amount of the LP tokens you'll get in exchange for your liquidity._
