# Create Pool

To create a liquidity pool you need to deploy a LP coin module on your account.

The LP coin can be any time deployed in your module and not initialized as coin, yet we suggest to copy our [LP Template](https://github.com/pontem-network/liquidswap-lp/blob/main/sources/lp.move) and just deploy it under your account.

{% embed url="https://gist.github.com/borispovod/ccda44afd615cf5aeea17b4ed2458414" %}
LP Template
{% endembed %}

_Don't forget to change account name on your named address: `liquidswap_lp.`_

After that, we can do a call to Router to create new `APTOS` / `BTC` liquidity pool on your account.

We can extend example from previous guide and add new function:

{% embed url="https://gist.github.com/borispovod/5caea21cc3777372ce18cbb03cd6cac0" %}
Create liquidity pool
{% endembed %}

#### Creation of stable pool

In same way you can create stable pool, just use `1` as `curve_type` argument.

{% embed url="https://gist.github.com/borispovod/a842a47f10a093d2591326b6ea85550c" %}
Create stable liquidity pool
{% endembed %}

### Adding Liquidity

Let's add liquidity to **our** pool:

{% embed url="https://gist.github.com/borispovod/1fbb2f7c3fd76a50371241ea89700c64" %}
Adding liquidity
{% endembed %}

As you see `add_liquidity` now allows to add liquidity in just created pool and after it deposit the remainders of both `X` and `Y` coins back on account, together with new `LP` coins.&#x20;

_During real world example we would recommend to deposit liquidity immidiatelly in the same transaction you created pool, also we would recommend to use slippage values for amount of LP coins you are getting in exchange for your liquidity._

### Adding Liquidity To Existing Pool

Let's say we want to add liquidity to existing `BTC / Aptos` created by Pontem team.

Only difference that we use `@liquidswap` address as pool address and different `LP`.

{% embed url="https://gist.github.com/borispovod/c6abe785342fd3a18059d7e155647dd4" %}
Adding liquidity to existing pool
{% endembed %}

_During real world example we would recommend to deposit liquidity immidiatelly in the same transaction you created pool, also we would recommend to use slippage values for amount of LP coins you are getting in exchange for your liquidity._
