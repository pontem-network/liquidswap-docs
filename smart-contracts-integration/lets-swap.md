# Let's Swap

**It's very important that during experiement with swaps be safe: always use correct slippage values, double check values, use only trusted pairs, double check external price sources.**&#x20;

During this guide we will use [Router](../../smart-contracts/#router) and try to swap Aptos coin to BTC.

_For sure, there is no real BTC, but for test, and we hope we would have wrapped one with LayerZero soon._

Let's create a new module and import router inside:

{% embed url="https://gist.github.com/borispovod/62ba3b63a335267221922db3eaa05891" %}
Basic module
{% endembed %}

Try to compile, you will get a warning, it's ok.&#x20;

Now let's create add more dependencies we will need during the example, and let's add basic entry function that we can call in transaction later:

{% embed url="https://gist.github.com/borispovod/1a1ec9b2ae21c0e0d75775f4fd9e6ecd" %}
Module with imports
{% endembed %}

Now let's withdraw `1000` Aptos coins from account and see query how BTC we can get:

{% embed url="https://gist.github.com/borispovod/c0d434a0c944009d83e18b2b7187b98f" %}
Example with querying amount out
{% endembed %}

The provided code above wouldn't still compile as we can't drop Aptos coins, in same time it already shows how much BTC we can get back for `1000` Aptos coins because of `get_amount_out` function.&#x20;

Now let's indeed do a swap and finally deposit swapped coins on our account:

{% embed url="https://gist.github.com/borispovod/019a3b99214845278d2339bef40aaf2c" %}
Final example
{% endembed %}

And it's finally just great, using  `swap_exact_coin_for_coin` we swap Aptos coins for BTC.

It's very simple, but what if someone would front-run us? Let's add at least some basic slippage check as minimum amount of btc we want to get out (after that we even can remove `get_amount_out`).

{% embed url="https://gist.github.com/borispovod/871dfeddbffba5396e5a605a46d7ef9c" %}
Final example with basic slippage
{% endembed %}

Well, it's all you need to do, in same way you can try yourself another function `router::swap_coin_for_exact_coin` (see [Router](../../smart-contracts/#router) docs).

\_What's important: always use the safety checks, never do the swaps without slippage provided offline, use external price feeds or oracles (even [Basic Oracle](basic-oracle.md) can be good example).
