# Simple Transfer Example

In this example, we will do a transfer on the relay chain initiated by a parachain.

Of course we could always call the `transfer` function of the balances pallet, but here we can
observe how the **holding register** is used.

</details>

What we want to see in this example is that through the process of transferring funds via XCM, we
execute the instructions `WithdrawAsset` and `DepositAsset`. By doing this, assets are taken out of
the sender account, placed into the holding register, and then deposited into the receiver account.

So for a moment, the assets exist only within the context of the holding registrar, and has been
completely removed from within the runtime state. This is a key point when thinking about cross
chain teleports.

<!-- slide:break -->

## Test

[Source
Code](https://github.com/paritytech/polkadot/blob/master/xcm/xcm-simulator/example/src/lib.rs#L234)

<details>
    <summary>Test: <code>fn withdraw_and_deposit()</code></summary>

```rust
/// Scenario:
/// A parachain transfers funds on the relay chain to another parachain account.
///
/// Asserts that the parachain accounts are updated as expected.
#[test]
fn withdraw_and_deposit() {
    MockNet::reset();
    let send_amount = 10;
    ParaA::execute_with(|| {
        let message = Xcm(vec![
            WithdrawAsset((Here, send_amount).into()),
            buy_execution((Here, send_amount)),
            DepositAsset { assets: All.into(), max_assets: 1, beneficiary: Parachain(2).into() },
        ]);
        // Send withdraw and deposit
        assert_ok!(ParachainPalletXcm::send_xcm(Here, Parent, message.clone()));
    });

    Relay::execute_with(|| {
        assert_eq!(
            relay_chain::Balances::free_balance(para_account_id(1)),
            INITIAL_BALANCE - send_amount
        );
        assert_eq!(relay_chain::Balances::free_balance(para_account_id(2)), send_amount);
    });
}
```

</details>

To run the test, execute:

```sh
cargo test --package xcm-simulator-example --lib -- tests::withdraw_and_deposit --exact --nocapture
```

### Output

```rust
running 1 test
instruction: WithdrawAsset(MultiAssets([MultiAsset { id: Concrete(MultiLocation { parents: 0, interior: Here }), fun: Fungible(10) }]))
holding before: Assets { fungible: {}, non_fungible: {} }
holding after: Assets { fungible: {Concrete(MultiLocation { parents: 0, interior: Here }): 10}, non_fungible: {} }

instruction: BuyExecution { fees: MultiAsset { id: Concrete(MultiLocation { parents: 0, interior: Here }), fun: Fungible(10) }, weight_limit: Unlimited }
holding before: Assets { fungible: {Concrete(MultiLocation { parents: 0, interior: Here }): 10}, non_fungible: {} }
holding after: Assets { fungible: {Concrete(MultiLocation { parents: 0, interior: Here }): 10}, non_fungible: {} }

instruction: DepositAsset { assets: Wild(All), max_assets: 1, beneficiary: MultiLocation { parents: 0, interior: X1(Parachain(2)) } }
holding before: Assets { fungible: {Concrete(MultiLocation { parents: 0, interior: Here }): 10}, non_fungible: {} }
holding after: Assets { fungible: {}, non_fungible: {} }

test tests::withdraw_and_deposit ... ok
```
