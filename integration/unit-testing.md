# Unit Testing

This part explains how to write unit tests during integration with Liquidswap Smart Contracts.

### Testing integration

To be able to run tests, you need to prepare the Liquidswap state in your test. There's a couple of utilities available to help you with that in - [source code](https://github.com/pontem-network/liquidswap/tree/main/sources/test\_helpers/test\_pool.move).

* `public fun initialize_liquidity_pool()`

It correctly initialize both Liquidswap and `LP<X, Y, Curve>` token of the Liquidswap. That process is pretty involved, so we provide this helper function to make it easier. Run it after the `genesis::setup()` call from Aptos Framework.

* `public fun mint_liquidity<X, Y, Curve>(lp_owner: &signer, coin_x: Coin<X>, coin_y: Coin<Y>): u64`

Adds `coin_x`, `coin_y` to the pool, and automatically deposits returning lp coins to the `lp_owner` account, handling the edge cases.

An example of the test would be:

{% embed url="https://gist.github.com/borispovod/41d239593acd6d1b8b1f4ad5110ae130" %}
Example of unit test
{% endembed %}

#### More examples

To learn Unit Tests more look at our Liquidswap [Unit Tests](https://github.com/pontem-network/liquidswap/tree/main/tests) and [Test Helpers](https://github.com/pontem-network/liquidswap/tree/main/sources/test\_helpers).
