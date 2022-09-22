# Create Pool

Now let's create a new pool utilizing Router. First of all, deploy your new own coin just for test:

{% embed url="https://gist.github.com/borispovod/a57e4de806dea23195412eb08f731f70" %}
Test coin for new pool
{% endembed %}

Extend the example from the previous guide and add a new function:

{% embed url="https://gist.github.com/borispovod/5caea21cc3777372ce18cbb03cd6cac0" %}
Create a new pool
{% endembed %}

The function above creates a new pool for your coin: `MyCoin/AptosCoin`. As Uncorrelated generic provided, the formula for the pool would be standard one: `x*y=k`.

It's very important to follow the rule about generic sorting as otherwise, the function would abort.

#### Creating a stable pool

In the same way, you can create a stable pool by use `Stable` generic:

{% embed url="https://gist.github.com/borispovod/1fbb2f7c3fd76a50371241ea89700c64" %}
Create stable pool
{% endembed %}

### Add liquidity to your pool

Let's add liquidity to new pool:

{% embed url="https://gist.github.com/borispovod/c6abe785342fd3a18059d7e155647dd4" %}
Adding liquidity to our new pool
{% endembed %}

As you can see above, `add_liquidity` allows to add liquidity to the newly created pool and then deposit the remainders of both `X` and `Y` coins back in the account, together with the new `LP` coins.

Similarly, you can add liquidity to any other pool: replace `MyCoin` and `AptosCoin` with other coins.

_When working with real-world examples, it's better to deposit liquidity immediately during the same transaction with which you create a pool. We also recommend using slippage values for the amount of the LP tokens you'll get in exchange for your liquidity._

### Adding liquidity to an existing pool

Let's say we want to add liquidity to an existing `BTC/Aptos` pool created by the Pontem team:

{% embed url="https://gist.github.com/borispovod/cc613eb511c4025641f18c7060fa9a60" %}
Adding liquidity to an existing pool
{% endembed %}

_When working with real-world examples, it's better to deposit liquidity immediately during the same transaction with which you create a pool. We also recommend using slippage values for the amount of the LP tokens you'll get in exchange for your liquidity._
