# Let's Swap

**During experiments with swaps, it's very important to be safe: always use correct slippage values, double-check values, use only trusted pairs, and compare external price sources.**

During this guide, we will use [Router](../smart-contracts/#router) and try to swap Aptos coin to test BTC.

Let's create a new module and import router inside:

{% embed url="https://gist.github.com/borispovod/62ba3b63a335267221922db3eaa05891" %}
Basic module
{% endembed %}

Try to compile the code above, you will get a warning, and it's ok.&#x20;

Now let's create add more dependencies we will need during the example, and let's add a basic entry function that we can call in a transaction later:

{% embed url="https://gist.github.com/borispovod/1a1ec9b2ae21c0e0d75775f4fd9e6ecd" %}
Module with imports
{% endembed %}

Let's withdraw `1000` Aptos coins from an account and see how BTC we can get in exchange:

{% embed url="https://gist.github.com/borispovod/c0d434a0c944009d83e18b2b7187b98f" %}
Example with querying amount out
{% endembed %}

The provided code above wouldn't still compile as we can't drop Aptos coins; at the same time, it already shows how much BTC we can get back for 1000 Aptos coins because of `get_amount_out` function.

Now let's indeed do a swap and finally deposit swapped coins on our account:

{% embed url="https://gist.github.com/borispovod/019a3b99214845278d2339bef40aaf2c" %}
Final example
{% endembed %}

And it's finally just great; using `swap_exact_coin_for_coin` we swap Aptos coins for BTC.

It's very simple, but what if someone would front-run us? Let's add some basic slippage checks for a minimum amount of BTC we want to get out (after that, we can even remove `get_amount_out`).

{% embed url="https://gist.github.com/borispovod/871dfeddbffba5396e5a605a46d7ef9c" %}
Final example with basic slippage
{% endembed %}

It's done. In the same way, you can try another function `router::swap_coin_for_exact_coin` (see [Router](../smart-contracts/#router) docs).

_What's important: always use the safety checks, never do the swaps without slippage provided offline, use external price feeds or oracles (even_ [_Basic Oracle_](basic-oracle.md) _can be good example)._
