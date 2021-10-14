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

Note that this instruction contains within it an XCM message which gets sent to the destination
chain.

These instructions do two simple things:

1. Pay for any weight for depositing a new asset into an account.
    > Note that we set a `weight_limit` of `Unlimited`. This is not very smart in general, but in
    > the early days of XCM, our end to end story for reporting weight isnt fully described. We
    > expect downstream teams to take sensible fees.
2. Depositing any and all assets which were reserve transferred.
    > Note that we use the `Wild(All)` assets filter, which will match to any assets which exist in
    > the holding register.

Within the executor logic, we append two additional instruction to the xcm:

```rust
let mut message = vec![ReserveAssetDeposited(assets), ClearOrigin];
message.extend(xcm.0.into_iter());
```

This will add on top of any instructions you want to execute, the instructions needed for the
downstream chain to recognize that the reserve asset has been deposited. This message is a
**trusted** message, and thus should only be respected by the destination chain if the origin is one
we respect. This can be other trusted parachains, or in the case of sending reserve tokens from the
relay chain to a parachain, parachains should always be able to trust the parent relay chain origin.

This all happens when accepting the downward message on the parachain.

https://github.com/paritytech/cumulus/blob/master/pallets/dmp-queue/src/lib.rs#L243

This logic is duplicated within the `xcm-simulator`, but ultimately, any message originating from
the relay chain is given an origin of `Parent`.



<!-- slide:break -->

## Reserve Transfer Test

Command:

```sh
cargo test --package xcm-simulator-example --lib -- tests::reserve_transfer --exact --nocapture
```

Output:

```
instruction: TransferReserveAsset { assets: MultiAssets([MultiAsset { id: Concrete(MultiLocation { parents: 0, interior: Here }), fun: Fungible(123) }]), dest: MultiLocation { parents: 0, interior: X1(Parachain(1)) }, xcm: Xcm([BuyExecution { fees: MultiAsset { id: Concrete(MultiLocation { parents: 1, interior: Here }), fun: Fungible(123) }, weight_limit: Unlimited }, DepositAsset { assets: Wild(All), max_assets: 1, beneficiary: MultiLocation { parents: 0, interior: X1(AccountId32 { network: Any, id: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0] }) } }]) }
origin: Some(MultiLocation { parents: 0, interior: X1(AccountId32 { network: Kusama, id: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0] }) })
holding before: Assets { fungible: {}, non_fungible: {} }
holding after: Assets { fungible: {}, non_fungible: {} }

instruction: ReserveAssetDeposited(MultiAssets([MultiAsset { id: Concrete(MultiLocation { parents: 1, interior: Here }), fun: Fungible(123) }]))
origin: Some(MultiLocation { parents: 1, interior: Here })
holding before: Assets { fungible: {}, non_fungible: {} }
holding after: Assets { fungible: {Concrete(MultiLocation { parents: 1, interior: Here }): 123}, non_fungible: {} }

instruction: ClearOrigin
origin: Some(MultiLocation { parents: 1, interior: Here })
holding before: Assets { fungible: {Concrete(MultiLocation { parents: 1, interior: Here }): 123}, non_fungible: {} }
holding after: Assets { fungible: {Concrete(MultiLocation { parents: 1, interior: Here }): 123}, non_fungible: {} }

instruction: BuyExecution { fees: MultiAsset { id: Concrete(MultiLocation { parents: 1, interior: Here }), fun: Fungible(123) }, weight_limit: Unlimited }
origin: None
holding before: Assets { fungible: {Concrete(MultiLocation { parents: 1, interior: Here }): 123}, non_fungible: {} }
holding after: Assets { fungible: {Concrete(MultiLocation { parents: 1, interior: Here }): 123}, non_fungible: {} }

instruction: DepositAsset { assets: Wild(All), max_assets: 1, beneficiary: MultiLocation { parents: 0, interior: X1(AccountId32 { network: Any, id: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0] }) } }
origin: None
holding before: Assets { fungible: {Concrete(MultiLocation { parents: 1, interior: Here }): 123}, non_fungible: {} }
holding after: Assets { fungible: {}, non_fungible: {} }

test tests::reserve_transfer ... ok
```
