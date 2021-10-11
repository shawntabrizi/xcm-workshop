# Buy Execution

XCM, like any other on-chain execution, uses the resources of the blockchain, and thus we expect
the message sender to pay fees fpr executing the message.

It is up to the recipient to determine what kinds of messages
should be allowed, what fees should be paid if any at all. This is done by
configuring a combination of the `Barrier` and `Trader` configurations of the `XcmExecutor`.
