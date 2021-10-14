# Barrier

The barrier is checked before any call is even executed, and acts as a filter to make sure that XCM
messages are even sane.

```rust
if let Err(_) =
    Config::Barrier::should_execute(&origin, &mut message, xcm_weight, &mut weight_credit)
{ return Outcome::Error(XcmError::Barrier) }
```

Then an implementation of `Barrier` like `AllowTopLevelPaidExecutionFrom` can check that somewhere
in the XCM message, you will be paying for the XCM via `BuyExecution`.

You can even allow "feeless" XCM by simply having a barrier like:

```rust
/// Allows execution from any origin that is contained in `T` (i.e. `T::Contains(origin)`) without any payments.
/// Use only for executions from trusted origin groups.
pub struct AllowUnpaidExecutionFrom<T>(PhantomData<T>);
impl<T: Contains<MultiLocation>> ShouldExecute for AllowUnpaidExecutionFrom<T> {
    fn should_execute<Call>(
        origin: &MultiLocation,
        _message: &mut Xcm<Call>,
        _max_weight: Weight,
        _weight_credit: &mut Weight,
    ) -> Result<(), ()> {
        ensure!(T::contains(origin), ());
        Ok(())
    }
}
```

In this case, we never check that `BuyExecution` was part of the XCM, yet we still allow the XCM,
because it is from some trusted origin.

<!-- slide:break -->

```rust
pub struct AllowTopLevelPaidExecutionFrom<T>(PhantomData<T>);
impl<T: Contains<MultiLocation>> ShouldExecute for AllowTopLevelPaidExecutionFrom<T> {
    fn should_execute<Call>(
        origin: &MultiLocation,
        message: &mut Xcm<Call>,
        max_weight: Weight,
        _weight_credit: &mut Weight,
    ) -> Result<(), ()> {
        ensure!(T::contains(origin), ());
        let mut iter = message.0.iter_mut();
        let i = iter.next().ok_or(())?;
        match i {
            ReceiveTeleportedAsset(..) |
            WithdrawAsset(..) |
            ReserveAssetDeposited(..) |
            ClaimAsset { .. } => (),
            _ => return Err(()),
        }
        let mut i = iter.next().ok_or(())?;
        while let ClearOrigin = i {
            i = iter.next().ok_or(())?;
        }
        match i {
            BuyExecution { weight_limit: Limited(ref mut weight), .. } if *weight >= max_weight => {
                *weight = max_weight;
                Ok(())
            },
            BuyExecution { ref mut weight_limit, .. } if weight_limit == &Unlimited => {
                *weight_limit = Limited(max_weight);
                Ok(())
            },
            _ => Err(()),
        }
    }
}
```
