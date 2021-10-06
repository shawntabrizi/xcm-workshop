# XCM Instructions

An XCM message is defined as:

```rust
pub struct Xcm<Call>(pub Vec<Instruction<Call>>);
```

**So XCM messages are just an ordered set of instructions.**

XCM is generic over `Call` which can be used to describe the set of dispatchable functions exposed
by the recipient of the message, specifically when making `Transact` instructions.

For example, if you want to call a specific pallet of another parachain, you will need to know their
`Call` enum in order to form the correct message. Even for the same pallet, this can vary from chain
to chain since each runtime can be configured differently.

This is why we standardize all the most useful instructions like `TransferAsset`, `QueryHolding`,
etc... It allows us to have a consistent language no matter which chain we talk to. But with an
instruction like `Transact`, we also have unlimited flexibility to call any "non-standard" functions
of another parachain.

## Example

Transferring a reserve asset to another parachain. [Source in `pallet-xcm`.](https://github.com/paritytech/polkadot/blob/master/xcm/pallet-xcm/src/lib.rs#L600)

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

<!-- slide:break -->

## XCM V2 Instructions

[Source Code](https://github.com/paritytech/polkadot/blob/master/xcm/src/v2/mod.rs#L206)

```rust
pub enum Instruction<Call> {
    WithdrawAsset(MultiAssets),
    ReserveAssetDeposited(MultiAssets),
    ReceiveTeleportedAsset(MultiAssets),
    QueryResponse { query_id: QueryId, response: Response, max_weight: u64 },
    TransferAsset { assets: MultiAssets, beneficiary: MultiLocation },
    TransferReserveAsset { assets: MultiAssets, dest: MultiLocation, xcm: Xcm<()> },
    Transact { origin_type: OriginKind, require_weight_at_most: u64, call: DoubleEncoded<Call> },
    HrmpNewChannelOpenRequest { sender: u32, max_message_size: u32, max_capacity: u32 },
    HrmpChannelAccepted { recipient: u32 },
    HrmpChannelClosing { initiator: u32, sender: u32, recipient: u32 },
    ClearOrigin,
    DescendOrigin(InteriorMultiLocation),
    ReportError { query_id: QueryId, dest: MultiLocation, max_response_weight: u64 },
    DepositAsset { assets: MultiAssetFilter, max_assets: u32, beneficiary: MultiLocation },
    DepositReserveAsset { assets: MultiAssetFilter, max_assets: u32, dest: MultiLocation, xcm: Xcm<()> },
    ExchangeAsset { give: MultiAssetFilter, receive: MultiAssets },
    InitiateReserveWithdraw { assets: MultiAssetFilter, reserve: MultiLocation, xcm: Xcm<()> },
    InitiateTeleport { assets: MultiAssetFilter, dest: MultiLocation, xcm: Xcm<()> },
    QueryHolding { query_id: QueryId, dest: MultiLocation, assets: MultiAssetFilter, max_response_weight: u64 },
    BuyExecution { fees: MultiAsset, weight_limit: WeightLimit },
    RefundSurplus,
    SetErrorHandler(Xcm<Call>),
    SetAppendix(Xcm<Call>),
    ClearError,
    ClaimAsset { assets: MultiAssets, ticket: MultiLocation },
    Trap(u64),
    SubscribeVersion { query_id: QueryId, max_response_weight: u64 },
    UnsubscribeVersion,
}
```
