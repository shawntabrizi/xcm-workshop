# Teleport Transfer Example

Finally, we will look at a teleport transfer.

This is similar to a reserve asset transfer, except the asset is completely moved to the new
consensus system, versus just minting a reserve asset. This means that teleports are only possible
between two systems which have complete trust with one another, or if the consensus system is simply
an extension of the other, such as with Kusama and Statemine.

To start we will call the instruction `WithdrawAsset` which you are already familiar with, which
will take assets and place them into the holding, but then after that we will call
`InitiateTeleport`:

```rust
InitiateTeleport { assets, dest, xcm } => {
    // We must do this first in order to resolve wildcards.
    let assets = self.holding.saturating_take(assets);
    for asset in assets.assets_iter() {
        Config::AssetTransactor::check_out(&dest, &asset);
    }
    let assets = Self::reanchored(assets, &dest)?;
    let mut message = vec![ReceiveTeleportedAsset(assets), ClearOrigin];
    message.extend(xcm.0.into_iter());
    Config::XcmSender::send_xcm(dest, Xcm(message)).map_err(Into::into)
},
```

Here you see we take the assets in holding, and call `check_out` on those assets. Then we will append additional instructions like `ReceiveTeleportedAsset` and `ClearOrigin`, which behaves just like the previous example.

<!-- slide:break -->

## On the Destination Chain

Now we will have a message coming to the destination chain which will say:

```rust
let mut message = Xcm(vec![
	ReceiveTeleportedAsset(assets),
	ClearOrigin,
	BuyExecution { fees, weight_limit: Limited(0) },
	DepositAsset { assets: Wild(all), max_assets, beneficiary }
]);
```

The key difference with a teleport transfer is that we check `IsTeleporter` before accepting any assets. Only trusted locations can initiate these kinds of message, otherwise we ignore them.

```rust
ReceiveTeleportedAsset(assets) => {
    let origin = self.origin.as_ref().ok_or(XcmError::BadOrigin)?;
    // check whether we trust origin to teleport this asset to us via config trait.
    for asset in assets.inner() {
        // We only trust the origin to send us assets that they identify as their
        // sovereign assets.
        ensure!(
            Config::IsTeleporter::filter_asset_location(asset, origin),
            XcmError::UntrustedTeleportLocation
        );
        // We should check that the asset can actually be teleported in (for this to be in error, there
        // would need to be an accounting violation by one of the trusted chains, so it's unlikely, but we
        // don't want to punish a possibly innocent chain/user).
        Config::AssetTransactor::can_check_in(&origin, asset)?;
    }
    for asset in assets.drain().into_iter() {
        Config::AssetTransactor::check_in(origin, &asset);
        self.holding.subsume(asset);
    }
    Ok(())
},
```
