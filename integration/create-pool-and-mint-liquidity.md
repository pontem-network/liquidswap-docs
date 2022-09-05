# Create Pool and Mint Liquidity

To create a liquidity pool you need to deploy a LP coin module on your account.

The LP coin can be any time deployed in your module and not initialized as coin, yet we suggest to copy our [LP Template](https://github.com/pontem-network/liquidswap-lp/blob/main/sources/lp.move) and just deploy it under your account.

{% embed url="https://gist.github.com/borispovod/ccda44afd615cf5aeea17b4ed2458414" %}

_Don't forget to change account name on your named address: `liquidswap_lp.`_

After that, we can do a call to Router to create new APTOS / BTC liquidity pool on your account.

We can extend example from previous guide and add  new function:



#### Creation of stable pool

