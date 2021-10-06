# XCM Executor

An XCM message is **just a message**... an encoded set of bytes, and does nothing by itself.

In order for a message to be processed, it must be interpreted by the XCM Executor which actually
executes operations based on the contents of the message, and maintains an internal state throughout
the execution.

## Holding Register

The XCM Executor can itself hold assets in a "virtual register", where it can manipulate and use
those assets. It can even transfer these assets over the boundaries of different consensus engines
(parachains, contracts, etc...).

<!-- slide:break -->

### XCM Executor State

[Source Code](https://github.com/paritytech/polkadot/blob/master/xcm/xcm-executor/src/lib.rs#L44)

```rust
/// The XCM executor.
pub struct XcmExecutor<Config: config::Config> {
    pub holding: Assets,
    pub origin: Option<MultiLocation>,
    pub original_origin: MultiLocation,
    pub trader: Config::Trader,
    pub error: Option<(u32, XcmError)>,
    pub total_surplus: u64,
    pub total_refunded: u64,
    pub error_handler: Xcm<Config::Call>,
    pub error_handler_weight: u64,
    pub appendix: Xcm<Config::Call>,
    pub appendix_weight: u64,
    _config: PhantomData<Config>,
}
```

### XCM Execution

```rust
/// Execute the XCM program fragment and report back the error and which instruction caused it,
/// or `Ok` if there was no error.
pub fn execute(&mut self, xcm: Xcm<Config::Call>) -> Result<(), ExecutorError> {
    let mut result = Ok(());
    for (i, instr) in xcm.0.into_iter().enumerate() {
        match &mut result {
            r @ Ok(()) => if let Err(e) = self.process_instruction(instr) {
                *r = Err(ExecutorError { index: i as u32, xcm_error: e, weight: 0 });
            },
            Err(ref mut error) => if let Ok(x) = Config::Weigher::instr_weight(&instr) {
                error.weight.saturating_accrue(x)
            },
        }
    }
    result
}
```
