---
sidebar_position: 1
---

# BeginBlocker and EndBlocker

:::note Synopsis
`BeginBlocker` and `EndBlocker` are optional methods module developers can implement in their module. They will be triggered at the beginning and at the end of each block respectively, when the [`BeginBlock`](../../learn/advanced/00-baseapp.md#beginblock) and [`EndBlock`](../../learn/advanced/00-baseapp.md#endblock) ABCI messages are received from the underlying consensus engine.
:::

:::note Pre-requisite Readings

* [Module Manager](./01-module-manager.md)

:::

## BeginBlocker and EndBlocker

`BeginBlocker` and `EndBlocker` are a way for module developers to add automatic execution of logic to their module. This is a powerful tool that should be used carefully, as complex automatic functions can slow down or even halt the chain. 

In 0.47.0, Prepare and Process Proposal were added that allow app developers to do arbitrary work at those phases, but they do not influence the work that will be done in BeginBlock. If an application required `BeginBlock` to execute prior to any sort of work is done then this is not possible today (0.50.0). 

When needed, `BeginBlocker` and `EndBlocker` are implemented as part of the [`HasBeginBlocker`, `HasABCIEndBlocker` and `EndBlocker` interfaces](./01-module-manager.md#appmodule). This means either can be left-out if not required. The `BeginBlock` and `EndBlock` methods of the interface implemented in `module.go` generally defer to `BeginBlocker` and `EndBlocker` methods respectively, which are usually implemented in `abci.go`.

The actual implementation of `BeginBlocker` and `EndBlocker` in `abci.go` are very similar to that of a [`Msg` service](./03-msg-services.md):

* They generally use the [`keeper`](./06-keeper.md) and [`ctx`](https://pkg.go.dev/context) to retrieve information about the latest state.
* If needed, they use the `keeper` and `ctx` to trigger state-transitions.
* If needed, they can emit [`events`](../../learn/advanced/08-events.md) via the `environments`'s `EventManager`.

A specific method (`UpdateValidators`) is available to return validator updates to the underlying consensus engine in the form of an [`[]appmodule.ValidatorUpdates`](https://github.com/cosmos/cosmos-sdk/blob/07151304e2ec6a185243d083f59a2d543253cb15/core/appmodule/v2/module.go#L87-L101). This is the preferred way to implement custom validator changes.

It is possible for developers to define the order of execution between the `BeginBlocker`/`EndBlocker` functions of each of their application's modules via the module's manager `SetOrderBeginBlocker`/`SetOrderEndBlocker` methods. For more on the module manager, click [here](./01-module-manager.md#manager).

See an example implementation of `BeginBlocker` from the `distribution` module:

```go reference
https://github.com/cosmos/cosmos-sdk/blob/v0.52.0-beta.1/x/distribution/keeper/abci.go#L13-L40
```

and an example of `EndBlocker` from the `gov` module:

```go reference
https://github.com/cosmos/cosmos-sdk/blob/v0.52.0-beta.1/x/gov/keeper/abci.go#L22
```

and an example implementation of `EndBlocker` with validator updates from the `staking` module:

```go reference
https://github.com/cosmos/cosmos-sdk/blob/v0.52.0-beta.1/x/staking/keeper/abci.go#L12-L17
```


<!-- TODO: leaving this here to update docs with core api changes  -->
