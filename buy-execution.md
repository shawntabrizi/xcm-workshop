# Buy Execution

XCM, like any other on-chain execution, uses the resources of the blockchain, and thus we expect the
message sender to pay fees for executing the message.


```rust
BuyExecution { fees, weight_limit } => {
	// There is no need to buy any weight is `weight_limit` is `Unlimited` since it
    // would indicate that `AllowTopLevelPaidExecutionFrom` was unused for execution
    // and thus there is some other reason why it has been determined that this XCM
    // should be executed.
    if let Some(weight) = Option::<u64>::from(weight_limit) {
        // pay for `weight` using up to `fees` of the holding register.
        let max_fee =
            self.holding.try_take(fees.into()).map_err(|_| XcmError::NotHoldingFees)?;
        let unspent = self.trader.buy_weight(weight, max_fee)?;
        self.holding.subsume_assets(unspent);
    }
    Ok(())
},
```

It is up to the recipient to determine what kinds of messages should be allowed, what fees should be
paid if any at all. This is done by configuring a combination of the `Barrier` and `Trader`
configurations of the `XcmExecutor`.

Take note of the note in `BuyExecution`. If the `weight_limit` is `Unlimited`, then the trader logic will be skipped, so it is important to have barriers set up which would prevent this.


<!-- slide:break -->

## Trader

The trader is a simple trait which tries to purchase `weight` using a `payment` asset.

```rust
/// Charge for weight in order to execute XCM.
///
/// A `WeightTrader` may also be put into a tuple, in which case the default behavior of
/// `buy_weight` and `refund_weight` would be to attempt to call each tuple element's own
/// implementation of these two functions, in the order of which they appear in the tuple,
/// returning early when a successful result is returned.
pub trait WeightTrader: Sized {
	/// Create a new trader instance.
	fn new() -> Self;

	/// Purchase execution weight credit in return for up to a given `fee`. If less of the fee is required
	/// then the surplus is returned. If the `fee` cannot be used to pay for the `weight`, then an error is
	/// returned.
	fn buy_weight(&mut self, weight: Weight, payment: Assets) -> Result<Assets, XcmError>;

	/// Attempt a refund of `weight` into some asset. The caller does not guarantee that the weight was
	/// purchased using `buy_weight`.
	///
	/// Default implementation refunds nothing.
	fn refund_weight(&mut self, _weight: Weight) -> Option<MultiAsset> {
		None
	}
}
```

This can be something as simple as a fixed price per weight, or using a DEX pallet to look up what the conversion factor should be.
