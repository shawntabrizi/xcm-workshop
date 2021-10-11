# Reserve Asset Deposited

To complete the reserve asset transfer, we need to trigger the appropriate logic on the parachain to
mint that reserve asset.

As mentioned earlier, we recieve the XCM from the DMP queue with the origin `Parent`. At this point, the XCM has the following instructions:

```rust
Xcm([
    ReserveAssetDeposited(assets),
    ClearOrigin,
    BuyExecution { fees, weight_limit: Unlimited },
    DepositAsset { assets, max_assets, beneficiary},
])
```

But just receiving a message like this is not enough for the chain to actually execute it. We should
**not** for example accept a message like this from a random parachain or user.

Instead, within the logic of `ReserveAssetDeposited` we check that we will accept the reserve
assets, and then place them into the holding register:

```rust
ReserveAssetDeposited(assets) => {
    let origin = self.origin.as_ref().ok_or(XcmError::BadOrigin)?;
    for asset in assets.drain().into_iter() {
        // Must ensure that we recognise the asset as being managed by the origin.
        ensure!(
            Config::IsReserve::filter_asset_location(&asset, origin),
            XcmError::UntrustedReserveLocation
        );
        self.holding.subsume(asset);
    }
    Ok(())
},
```

<!-- slide:break -->

The `IsReserve` trait is configurable in the `XcmConfig`. A common pattern is to use the helper
function for supporting native assets of the origin chain. That is DOT for Polkadot, KSM for Kusama,
etc:

```rust
/// Accepts an asset iff it is a native asset.
pub struct NativeAsset;
impl FilterAssetLocation for NativeAsset {
	fn filter_asset_location(asset: &MultiAsset, origin: &MultiLocation) -> bool {
		matches!(asset.id, Concrete(ref id) if id == origin)
	}
}
```

So here we can see we will only accept assets which are `Concrete` and match the origin. From our
example logs we see:

```sh
instruction: ReserveAssetDeposited(MultiAssets([MultiAsset { id: Concrete(MultiLocation { parents: 1, interior: Here }), fun: Fungible(123) }]))
origin: Some(MultiLocation { parents: 1, interior: Here })
```

So we are doing a reserve asset deposit of `Concrete(MultiLocation { parents: 1, interior: Here })`
from the origin `Some(MultiLocation { parents: 1, interior: Here })`, which match and pass the
filter! This being successful will now place the reserve asset into the holding registrar of the
parachain XCM Executor.

The next instruction is `ClearOrigin` which is important since the sender defines all the subsequent
instructions, and it could be an elevation of privledges to keep the origin around.
