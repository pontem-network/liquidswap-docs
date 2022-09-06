# Create Pool

You must deploy an LP coin module on your account to create a liquidity pool.

We suggest you copy our [LP Template ](https://github.com/pontem-network/liquidswap-lp/blob/main/sources/lp.move)and just deploy it under your account.

{% embed url="https://gist.github.com/borispovod/ccda44afd615cf5aeea17b4ed2458414" %}
LP Template
{% endembed %}

_Don't forget to replace `liquidswap_lp` in module declaration with your named address after deployment._

After that, we can do a call to Router to create a new `APTOS` / `BTC` liquidity pool on your account.

Extend the example from the previous guide and add a new function:

{% embed url="https://gist.github.com/borispovod/5caea21cc3777372ce18cbb03cd6cac0" %}
Create liquidity pool
{% endembed %}

#### Creation of stable pool

In the same way, you can create a stable pool but use `1` as the `curve_type` argument.

{% embed url="https://gist.github.com/borispovod/a842a47f10a093d2591326b6ea85550c" %}
Create stable liquidity pool
{% endembed %}

### Add liquidity to your pool

Let's add liquidity to **our** pool:

{% embed url="https://gist.github.com/borispovod/1fbb2f7c3fd76a50371241ea89700c64" %}
Adding liquidity
{% endembed %}

As you can see above, `add_liquidity`  allows to add liquidity in the just created pool, and after it deposits the remainders of both `X` and `Y` coins back on account, together with new `LP` coins.&#x20;

_During real-world examples, we would recommend depositing liquidity immediately in the same transaction you created pool. Also, we recommend using slippage values for the amount of LP coins you get in exchange for your liquidity._

### Adding Liquidity To Existing Pool

Let's say we want to add liquidity to existing `BTC / Aptos` created by Pontem team.

The only difference is that we use `@liquidswap` address as the pool address and a different LP.

{% embed url="https://gist.github.com/borispovod/c6abe785342fd3a18059d7e155647dd4" %}
Adding liquidity to existing pool
{% endembed %}

_During real-world examples, we would recommend depositing liquidity immediately in the same transaction you created pool. Also, we recommend using slippage values for the amount of LP coins you get in exchange for your liquidity._
