# Let's Swap

**Always practice solid risk management rules when experimenting with swaps: use the correct slippage values and trusted currency pairs, double-check the numbers before confirming, and compare external price sources.**

Throughout this guide, we will use [Router](../smart-contracts/#router) and try to swap Aptos coins to test BTC.

Let's create a new module and import the router into it:

{% embed url="https://gist.github.com/borispovod/62ba3b63a335267221922db3eaa05891" %}
Basic module
{% endembed %}

As you try to compile the code above, you will get a warning; don't worry, it's fine.&#x20;

Now let's add the dependencies we will need for this experiment, as well as a basic entry function that we can call in a transaction later:

{% embed url="https://gist.github.com/borispovod/1a1ec9b2ae21c0e0d75775f4fd9e6ecd" %}
Module with imports
{% endembed %}

Let's withdraw `1000` Aptos coins from an account and see how many BTC we can get in exchange:

{% embed url="https://gist.github.com/borispovod/c0d434a0c944009d83e18b2b7187b98f" %}
Example with querying amount out
{% endembed %}

The provided code above wouldn't still compile as we can't drop Aptos coins; at the same time, it already shows how much BTC we can get back for 1000 Aptos coins because of `get_amount_out` function.

Now let's indeed do a swap and finally deposit the swapped coins on our account:

{% embed url="https://gist.github.com/borispovod/019a3b99214845278d2339bef40aaf2c" %}
Final example
{% endembed %}

Everything worked out well; using `swap_exact_coin_for_coin,` we have swapped Aptos coins for BTC.

This looks simple, but what if someone decided to front-run us? Let's add some basic slippage checks for the minimum amount of BTC we want to receive (afterwards we can remove `get_amount_out`).

{% embed url="https://gist.github.com/borispovod/871dfeddbffba5396e5a605a46d7ef9c" %}
Final example with basic slippage
{% endembed %}

All done. In the same way, you can try another function: `router::swap_coin_for_exact_coin` (see [Router](../smart-contracts/#router) docs).

_An important note regarding decentralized AMM securities and user transactions:_

* _Always use safety checks._
* _Never swap tokens without a slippage value provided offline._
* _Use external price feeds or oracles (even the_ [_Basic Oracle_](basic-oracle.md) _can be a good option)._
