# Flashloans

The Liquidswap allows to loan liquidity from pools, use that liquidity in any 3rd party DeFi protocol or even in another Liquidswap pool itself, and return liquidity to the pool by paying the pool's standard fee.

The loan must happen in one transaction **(and it's impossible to loan and then return the loan in a separate transaction!)** and be returned in one transaction, which means Liquidswap liquidity never drains.

### Smart Contracts

Source code - .[/sources/swap/liquidity\_pool.move](https://github.com/pontem-network/liquidswap/blob/main/sources/swap/liquidity\_pool.move)

The idea of Flashloan is the **"Hot Potato"** concept; read more in [the medium article](https://medium.com/@borispovod/move-hot-potato-pattern-bbc48a48d93c); in a nutshell: because Move language structs have abilities, we can create a such structure that can't be copied, dropped, stored, but only can be destructed.&#x20;

In same time Move language allows to objects only in modules where initial structure defined. &#x20;

So by creating and returning the Flashloan object to the loaner, we force the loaner to return the loan together with the Flashloan object by calling the function used to pay the loan.

```rust
struct Flashloan<phantom X, phantom Y, phantom Curve> {
    x_loan: u64,
    y_loan: u64
}
```

Functions to implement flashloan in your code:

**Most of the functions of the specific LiquidityPool\<X, Y, Curve> wouldn't work during flashloan on that pool.**

The list of flashloan functions

* `flashloan<X, Y, Curve>(x_loan, y_loan)` - to loan X and Y coins from the pool.
  * **Generics X and Y must be sorted.**
  * `x_loan` - amount of X coins to loan.
  * `y_loan` - amount of Y coins to loan.
  * Returns `Coin<X>`, `Coin<Y>`and `Flashloan<X,Y, Curve>` object. &#x20;
* `pay_flashloan<X, Y, Curve>(x_in, y_in, loan):`
  * **Generics X and Y must be sorted.**
  * `x_in` - `Coin<X>` to return to pool.
  * `y_in` - `Coin<Y>` to return to the pool.

### Example

{% embed url="https://gist.github.com/borispovod/1269f2a70e0963425ca62892f521f667" %}
Flashloan Example
{% endembed %}

