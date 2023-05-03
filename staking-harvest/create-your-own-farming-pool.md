# Create your own farming pool

**Important:** It is crucial to have a comprehensive understanding of the tasks involved, as there are inherent risks, including the potential loss of funds.

If you are interested in creating a new farming pool on Liquidswap and providing rewards, this is the page you should study. The current example demonstrates the process using the Aptos CLI, but alternative methods can be employed with the Aptos SDK or a custom user interface (note that a default interface is not currently provided and there are no plans to develop one at this time).

## Preparation

Before creating a farming pool, if you want it to be visible on the Liquidswap DApp, please [contact us](https://t.me/pontemnetworkchat) first and communicate your intentions. Otherwise, we will not be able to whitelist your reward pool.

To create a farming pool, you will need:

* [Aptos CLI](https://github.com/aptos-labs/aptos-core/releases)
* A new account funded with Aptos to cover gas fees and containing the reward coins you plan to use for the pool.
* **Optional:** If you want to enable users to stake NFTs for a staking boost, you will also need information about the staking collection:
  2. Address of the collection creator
  3. Name of the collection

Before creating the pool, prepare a folder that will store the Aptos profile containing the private key from the account you plan to use for pool creation.

If you're unsure how to do this, simply create a new directory and run the `aptos init` command there. As a result, you can either create a new account and transfer tokens to it or use the private key from an existing account.

### Pool Creation

To create a pool without an NFT collection booster, use the following command from the directory containing your Aptos CLI profile.&#x20;

Make sure to replace the placeholder values with your own information:

```sh
aptos move run --function-id 0xb247ddeee87e848315caf9a33b8e4c71ac53db888cb88143d62d2370cca0ead2::scripts::register_pool \
  --args u64:0 \ # Amount of rewards.
  --args u64:0 \ # Duration of the pool in seconds.
  --type-args "" \ # Coin type of coin which will be used to stake.
  --type-args "" \ # Reward coin type, used as reward, e.g. 0x1::aptos_coin::AptosCoin.
```

Example of a stake coin type if you want to allow people to stake lzUSDC / APT LP coins:

```
0x05a97986a9d031c4567e15b797be516910cfcb4156312482efc6a19c0a30c948::lp_coin::LP<0xf22bede237a07e121b56d91a491eb7bcdfd1f5907926a9e58338f964a01b17fa::asset::USDC, 0x1::aptos_coin::AptosCoin, 0x190d44266241744264b964a37b8f09863167a12d3e70cda39376cfb4e3561e12::curves::Uncorrelated>
```

If you want to create a pool with NFT collection use another command:

```sh
aptos move run --function-id 0xb247ddeee87e848315caf9a33b8e4c71ac53db888cb88143d62d2370cca0ead2::scripts::register_pool_with_collection \
  --args u64:0 \ # Amount of rewards.
  --args u64:0 \ # Duration of the pool in seconds.
  --args address:0x1 \ # Creator of the collection.
  --args string:"" \ # Name of the collection.
  --args u128:0 \ # Booster, from 1 to 100.
  --type-args "" \ # Coin type of coin which will be used to stake.
  --type-args "" \ # Reward coin type, used as reward, e.g. 0x1::aptos_coin::AptosCoin.
```

After creating the new pool, please provide us with the following information:

* A link to the transaction in the blockchain explorer.
* Account address that created the pool (so we can verify).\
