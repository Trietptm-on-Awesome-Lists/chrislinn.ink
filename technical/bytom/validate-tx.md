# 验证交易

`func ValidateTx(tx *bc.Tx, block *bc.Block) (*GasState, error)` 对交易进行验证

交易除了原生 types 层之外，有个 map 层的封装，便于 验证

```
type validationState struct {
    block     *bc.Block
    tx        *bc.Tx
    gasStatus *GasState
    entryID   bc.Hash           // The ID of the nearest enclosing entry
    sourcePos uint64            // The source position, for validate ValueSources
    destPos   uint64            // The destination position, for validate ValueDestinations
    cache     map[bc.Hash]error // Memoized per-entry validation results
}
```

`gasStatus := &GasState{GasValid: false}` 一开始是 false，然后随着验证逐步更新， 如果最终有效  GasValid  才为 true
