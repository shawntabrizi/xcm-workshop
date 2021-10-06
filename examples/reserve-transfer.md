# Reserve Transfer Example

In this example, we will move some of the relay chain currency into a parachain via a reserve
transfer. This is exactly what third party parachains do on Kusama to transfer KSM to their chain.

For this, we will take advantage of the existing helper extrinsic in `pallet-xcm`:
[source](https://github.com/paritytech/polkadot/blob/master/xcm/pallet-xcm/src/lib.rs#L574).

## Breakdown

The complete execution of this XCM message will take place on both the relay chain and the parachain
where the asset is being transferred to.

The message starts on the Relay Chain from the user who wants to reserve transfer funds:

```rust
let mut message = Xcm(vec![TransferReserveAsset {
	assets,
	dest,
	xcm: Xcm(vec![
		BuyExecution { fees, weight_limit: Unlimited },
		DepositAsset { assets: Wild(All), max_assets, beneficiary },
	]),
}]);
```
