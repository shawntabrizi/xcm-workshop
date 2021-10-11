# Glossary

This page will help familiarize you with new terminology from within the XCM world.

## Consensus System

From the [`xcm-format`](https://github.com/paritytech/xcm-format) specification:

> A chain, contract or other global, encapsulated, state machine singleton. It can be any
> programmatic state-transition system that exists within consensus which can send/receive
> datagrams. May be specified by a MultiLocation value (though not all such values identify a
> Consensus System). Examples include The Polkadot Relay chain, The XDAI Ethereum PoA chain, The
> Ethereum Tether smart contract.

## DMP

An acronym for "Downward Message Passing". This is a polkadot specific protocol for how message are
sent from a parent relay chain to a child parachain. Hence the message goes "downward".

## Reserve Asset

A reserve asset transfer is a specific kind of asset transfer where assets are moved under control
of a sovereign account of a consensus system, and an equivalent amount of those assets are minted
into that consensus system.

These newly minted assets on the new consensus system are backed by the actual assets held by the
original asset creator, just like a reserve currency.

This is how most parachains will transfer assets between one another.

See [Teleported Asset](#teleported-asset)

## Sovereign Account

From the [`xcm-format`](https://github.com/paritytech/xcm-format) specification:

> An account controlled by a particular Consensus System, within some other Consensus System. There
> may be many such accounts or just one. If many, then this assumes and identifies a unique primary
> account.

## System Chain

A system chain is a parachain which acts as an extension of the relay chain.

Notice that relay chains like Kusama and Polkadot have many functionalities beyond simply managing
parachains. Things like governance, balances, staking, identity, and more. Ideally all of these
functions will be migrated to system chains which are dedicated parachains which will offload these
functions from the relay chain.

It is important to note that system chains do not have any of their own token economics or sovreignty. They use
the token and governance of the relay chain, and should basically be considered the same chain.

## Teleported Asset

A teleport is a specific kind of asset transfer where assets are completely removed from one
consensus system and placed into another system. Hence the assets are "teleported".

This operation should only be performed by trusted parties, since management of the assets
completely leave the scope of the original consensus system.

For example, because Statemine is simply a logical extension of Kusama, KSM can be teleported from
Kusama to Statemine. Owning KSM on Statemine should be considered the same as owning it on Kusama.

See [Reserve Asset](#reserve-asset)

## UMP

An acronym for "Upward Message Passing". This is a polkadot specific protocol for how messages are
sent from a child parachain to the parent relay chain. Hence the message goes "upward".

## XCM

An acronym for "Cross-Consensus Messages". The use of "consensus" here refers to the fact that these
messages can be used to interact across different kinds of applications, such as parachains, smart
contracts, or any other consensus engines. XCM is design to be platform agnostic.

## XCMP

An acronym for "Cross-Chain Message Passing". This is a Polkadot specific protocol for how messages
are sent between sibling parachains.

XCMP is still under development, and it's current iteration is called XCMP Lite (or HRMP for
"Horizontal Relay Message Passing"). The main difference between XCMP Lite and the final production
version of XCMP is that XCMP Lite currently uses the relay chain to facilitate the passing of
messages between parachains, whereas our goal for XCMP is to not include the relay chain at all for
such messages.

See: [DMP](#dmp) and [UMP](#ump)
