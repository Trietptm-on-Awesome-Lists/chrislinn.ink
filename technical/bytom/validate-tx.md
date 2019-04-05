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

先进行一些基本检查，比如 checkStandardTx 检查是否是标准交易（P2SH, P2PKH, OP_TRUE, OP_FAIL）

然后 checkValid 检查输入输出是否合法(输入输出是否平衡，是否进行了正确的引用)，以及 gas 是否足够

validationState 中，entryID 为待校验的元素，比如 coinbase, spend, issue, retire, mux 等等， 其中 mux 作为一个连接器 连接 输入与输出

cache 则是对之前校验过的 entry 的 error 进行缓存，避免重复校验

`func checkValid(vs *validationState, e bc.Entry) ` 中反复有类似 `vs2 := *vs`
的语句出现是为了防止 浅拷贝 造成修改

`vm.Verify` 涉及到 [虚拟机验证](vm-valify.md)，

