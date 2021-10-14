# XCM Config

The XCM Executor is very modular and configurable. We use many different traits, all of which can be
implemented in different ways to be able to allow full flexibility for users to handle XCM messages.

For most of these, there are even more generic builder structs that can be used to easily implement
these configurations.

Here is an example implementation from the Kusama runtime:

```rust
pub struct XcmConfig;
impl xcm_executor::Config for XcmConfig {
	type Call = Call;
	type XcmSender = XcmRouter;
	type AssetTransactor = LocalAssetTransactor;
	type OriginConverter = LocalOriginConverter;
	type IsReserve = ();
	type IsTeleporter = TrustedTeleporters;
	type LocationInverter = LocationInverter<Ancestry>;
	type Barrier = Barrier;
	type Weigher = FixedWeightBounds<BaseXcmWeight, Call, MaxInstructions>;
	// The weight trader piggybacks on the existing transaction-fee conversion logic.
	type Trader = UsingComponents<WeightToFee, KsmLocation, AccountId, Balances, ToAuthor<Runtime>>;
	type ResponseHandler = XcmPallet;
	type AssetTrap = XcmPallet;
	type AssetClaims = XcmPallet;
	type SubscriptionService = XcmPallet;
}
```

```rust
parameter_types! {
	/// The location of the KSM token, from the context of this chain. Since this token is native to this
	/// chain, we make it synonymous with it and thus it is the `Here` location, which means "equivalent to
	/// the context".
	pub const KsmLocation: MultiLocation = Here.into();
	/// The Kusama network ID. This is named.
	pub const KusamaNetwork: NetworkId = NetworkId::Kusama;
	/// Our XCM location ancestry - i.e. what, if anything, `Parent` means evaluated in our context. Since
	/// Kusama is a top-level relay-chain, there is no ancestry.
	pub const Ancestry: MultiLocation = Here.into();
	/// The check account, which holds any native assets that have been teleported out and not back in (yet).
	pub CheckAccount: AccountId = XcmPallet::check_account();
}

/// The canonical means of converting a `MultiLocation` into an `AccountId`, used when we want to determine
/// the sovereign account controlled by a location.
pub type SovereignAccountOf = (
	// We can convert a child parachain using the standard `AccountId` conversion.
	ChildParachainConvertsVia<ParaId, AccountId>,
	// We can directly alias an `AccountId32` into a local account.
	AccountId32Aliases<KusamaNetwork, AccountId>,
);

/// Our asset transactor. This is what allows us to interest with the runtime facilities from the point of
/// view of XCM-only concepts like `MultiLocation` and `MultiAsset`.
///
/// Ours is only aware of the Balances pallet, which is mapped to `KsmLocation`.
pub type LocalAssetTransactor = XcmCurrencyAdapter<
	// Use this currency:
	Balances,
	// Use this currency when it is a fungible asset matching the given location or name:
	IsConcrete<KsmLocation>,
	// We can convert the MultiLocations with our converter above:
	SovereignAccountOf,
	// Our chain's account ID type (we can't get away without mentioning it explicitly):
	AccountId,
	// We track our teleports in/out to keep total issuance correct.
	CheckAccount,
>;

/// The means that we convert an the XCM message origin location into a local dispatch origin.
type LocalOriginConverter = (
	// A `Signed` origin of the sovereign account that the original location controls.
	SovereignSignedViaLocation<SovereignAccountOf, Origin>,
	// A child parachain, natively expressed, has the `Parachain` origin.
	ChildParachainAsNative<parachains_origin::Origin, Origin>,
	// The AccountId32 location type can be expressed natively as a `Signed` origin.
	SignedAccountId32AsNative<KusamaNetwork, Origin>,
	// A system child parachain, expressed as a Superuser, converts to the `Root` origin.
	ChildSystemParachainAsSuperuser<ParaId, Origin>,
);

parameter_types! {
	/// The amount of weight an XCM operation takes. This is a safe overestimate.
	pub const BaseXcmWeight: Weight = 1_000_000_000;
	/// Maximum number of instructions in a single XCM fragment. A sanity check against weight
	/// calculations getting too crazy.
	pub const MaxInstructions: u32 = 100;
}

/// The XCM router. When we want to send an XCM message, we use this type. It amalgamates all of our
/// individual routers.
pub type XcmRouter = (
	// Only one router so far - use DMP to communicate with child parachains.
	xcm_sender::ChildParachainRouter<Runtime, XcmPallet>,
);

parameter_types! {
	pub const Kusama: MultiAssetFilter = Wild(AllOf { fun: WildFungible, id: Concrete(KsmLocation::get()) });
	pub const KusamaForStatemint: (MultiAssetFilter, MultiLocation) = (Kusama::get(), Parachain(1000).into());
}
pub type TrustedTeleporters = (xcm_builder::Case<KusamaForStatemint>,);

/// The barriers one of which must be passed for an XCM message to be executed.
pub type Barrier = (
	// Weight that is paid for may be consumed.
	TakeWeightCredit,
	// If the message is one that immediately attemps to pay for execution, then allow it.
	AllowTopLevelPaidExecutionFrom<Everything>,
	// Messages coming from system parachains need not pay for execution.
	AllowUnpaidExecutionFrom<IsChildSystemParachain<ParaId>>,
);
```

<!-- slide:break -->

```rust
/// The trait to parameterize the `XcmExecutor`.
pub trait Config {
    /// The outer call dispatch type.
    type Call: Parameter + Dispatchable<PostInfo = PostDispatchInfo> + GetDispatchInfo;
    /// How to send an onward XCM message.
    type XcmSender: SendXcm;
    /// How to withdraw and deposit an asset.
    type AssetTransactor: TransactAsset;
    /// How to get a call origin from a `OriginKind` value.
    type OriginConverter: ConvertOrigin<<Self::Call as Dispatchable>::Origin>;
    /// Combinations of (Location, Asset) pairs which we trust as reserves.
    type IsReserve: FilterAssetLocation;
    /// Combinations of (Location, Asset) pairs which we trust as teleporters.
    type IsTeleporter: FilterAssetLocation;
    /// Means of inverting a location.
    type LocationInverter: InvertLocation;
    /// Whether we should execute the given XCM at all.
    type Barrier: ShouldExecute;
    /// The means of determining an XCM message's weight.
    type Weigher: WeightBounds<Self::Call>;
    /// The means of purchasing weight credit for XCM execution.
    type Trader: WeightTrader;
    /// What to do when a response of a query is found.
    type ResponseHandler: OnResponse;
    /// The general asset trap - handler for when assets are left in the Holding Register at the
    /// end of execution.
    type AssetTrap: DropAssets;
    /// The handler for when there is an instruction to claim assets.
    type AssetClaims: ClaimAssets;
    /// How we handle version subscription requests.
    type SubscriptionService: VersionChangeNotifier;
}
```
