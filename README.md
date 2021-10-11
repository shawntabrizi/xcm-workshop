# XCM Workshop

This is an interactive workshop walking your through Polkadot's XCM (cross-consensus messages) and
XCMP (cross-chain message passing).

The XCM implementation is built on top of many layers of abstractions and configurations. In order
to understand how XCM works, you will need to jump between the XCM format, the XCM Executor, XCM
Builder, XCM Configuration and more!

This workshop will walk you through those points and peel the layers of the onion.

We will focus on the 3 main types of balance transfers:

* A simple single chain transfer initiated by a parachain.
* A reserve backed transfer from the relay chain to a parachain.
* A teleport transfer from the relay chain to a trusted system chain.

## Resources

Here are some great resources to read to learn about XCM before jumping into this workshop:

* Gav's 3 Part XCM Overview:
    1. https://medium.com/polkadot-network/xcm-the-cross-consensus-message-format-3b77b1373392
    2. https://medium.com/polkadot-network/xcm-part-ii-versioning-and-compatibility-b313fc257b83
    3. https://medium.com/polkadot-network/xcm-part-iii-execution-and-error-management-ceb8155dd166

* XCM Format Specification: https://github.com/paritytech/xcm-format

* Substrate Office Hours - XCM AMA: https://www.youtube.com/watch?v=cS8GvPGMLS0

## Contributing

This workshop was designed for the Sub0 Developer Conference taking place Oct 13-14 of 2021.

Substrate, Polkadot, XCM, and XCMP are all quickly developing products, and as such, documentation
can go out of date.

Feel free to open an issue or pull request if you find any issues with these pages.
