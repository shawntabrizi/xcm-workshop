# Buy Execution

XCM, like any other on-chain execution, uses the resources of the blockchain, and thus we expect the
message sender to pay fees for executing the message.


```rust
BuyExecution { fees, weight_limit } => {
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
