# Oracle Module

`x/oracle` provides a way to receive external information(real world price, events from other chains, etc.) with validators' vote. Each validator make transaction which contains those informations, and Oracle aggregates them until the supermajority signed on it. After then, Oracle sends the information to the actual module that processes the information, and prune the votes from the state.

## Integration

See `x/oracle/oracle_test.go` for the code that using Oracle

To use Oracle in your module, first define a `payload`. It should implement `oracle.Payload` and contain nessesary information for your module. Including nonce is recommended.

```go
type MyPayload struct {
    Data  int
    Nonce int
}
```

When you write a payload, its `.Route()` should return same route with your module is registered on the router. It is because `oracle.Msg` inherits `.Route()` from its embedded payload and it should be handled on the user modules.

Then route every incoming `oracle.Msg` to `oracle.Keeper.Handler()` with the function that implements `oracle.Handler`.

```go
func NewHandler(keeper Keeper) sdk.Handler {
    return func(ctx sdk.Context, msg sdk.Msg) sdk.Result {
        switch msg := msg.(type) {
        case oracle.Msg: 
            return keeper.oracle.Handle(ctx sdk.Context, p oracle.Payload) sdk.Error {
                switch p := p.(type) {
                case MyPayload:
                    return handleMyPayload(ctx, keeper, p)
                }
            }
        }
    }
}
```

In the previous example, the keeper has an `oracle.Keeper`. `oracle.Keeper`s are generated by `NewKeeper`.

```go
func NewKeeper(key sdk.StoreKey, cdc *codec.Codec, valset sdk.ValidatorSet, supermaj sdk.Dec, timeout int64) Keeper {
    return Keeper {
        cdc: cdc,
        key: key,
    
        // ValidatorSet to get validators infor
        valset: valset,

        // The keeper will pass payload
        // when more than 2/3 signed on it
        supermaj: supermaj,
        // The keeper will prune votes after 100 blocks from last sign
        timeout: timeout,
    }
}
```

Now the validators can send `oracle.Msg`s with `MyPayload` when they want to witness external events. 