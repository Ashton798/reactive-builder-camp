# D2 读懂费用：Lasna 系统合约参数

查询时间：2026-11-17（北京时间）。这些是查询时的链上值，参数以后可能变化。查询为只读，不需要签名或支付 Gas。

## 三条 cast 命令与输出

```powershell
cast call 0x8888888888888888888888888888888888888888 "_gasPriceCoeffPer1000()(uint256)" --rpc-url https://lasna-rpc.rnk.dev/
# 1000

cast call 0x8888888888888888888888888888888888888888 "_extraGas()(uint256)" --rpc-url https://lasna-rpc.rnk.dev/
# 100000 [1e5]

cast call 0x8888888888888888888888888888888888888888 "_maxChargeGas()(uint256)" --rpc-url https://lasna-rpc.rnk.dev/
# 50000 [5e4]
```

第一个数除以 1000 得到计费系数 1。第二个数代表固定附加 100000 gas。第三个数是付款结算可使用的 gas 预算上限，不是 react() 的费用上限，也不直接代入计费公式。

## 1. 一次 react() 的费用怎么算？

费用 = 计费 gas 单价 × 系数 ×（实际消耗 gas + 固定附加 gas）。计费单价取交易 gas 价格与区块 base fee 中较高者。本次查到系数为 1，固定附加量 100000 gas。

打个比方：网络像帮我们办事的服务员，除了干活的计算量，还要支付通知和记账的固定成本。如果假设计费单价是 100 gwei，实际消耗 50000 gas，计算为 100 gwei × 1 ×（50000 + 100000）= 0.015 lREACT。例子中的单价和消耗是为了说明，实际费用随交易变化。

## 2. balance、reserves、debt 分别是什么？

- balance：合约自己账户里的钱，类似钱包里的钱。
- reserves：预先存放在系统合约用于付费的额度，类似已充值话费。
- debt：已经发生但未结清的费用，类似欠费。

只要 debt 大于 0，系统就会暂停向合约派发新的 react()。使用系统合约 depositTo(合约地址) 充值时先还债，剩余才变成预付款。直接给 Reactive 合约转钱只增加 balance，不会自动清偿已有债务。

我们的 Demo 测试事件 15 后，查询时合约 balance 是 0.01 lREACT、debt 是 0.0181705 lREACT；这解释了为什么账户有余额仍会暂停。数值是当时的快照。

## 3. react() 回滚还收费吗？

收费。虽然业务结果回滚，网络仍然做了计算，因此按规则计费。设计合约时要校验事件来源、主题和数据格式；对不符合条件的事件直接返回，不要使用回滚当筛选器；设置执行次数上限，并监控预付款和欠费。我们的 Demo 使用 onlySystem、数据校验和最多 5 次处理。

参考：[Reactive 官方 Economy 文档](https://dev.reactive.network/economy) 和 [Reactive Contracts 文档](https://dev.reactive.network/reactive-contracts)。
