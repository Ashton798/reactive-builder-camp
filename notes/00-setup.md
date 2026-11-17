# 第 0 周：环境与 Omni 上下文

- Foundry Forge、Cast、Anvil 1.9.0 已完成版本检查。
- 测试钱包和测试币：待完成。
- Lasna RPC: https://lasna-rpc.rnk.dev/，Chain ID: 5318007，币种：lREACT。
- 本地 Demo 编译通过，6 项模拟测试通过；真实链上自动化尚未验证。

## 给 AI 的上下文

使用 Reactive Network Omni 新版。参考 https://dev.reactive.network/reactive-contracts 、https://dev.reactive.network/callback-contracts 、https://dev.reactive.network/economy 以及 https://github.com/Reactive-Network/reactive-smart-contract-demos/tree/omni/src/micro-demos 。

新合约不使用旧版 vm 修饰符、ReactVM 或 emit Callback；使用系统合约 0x8888888888888888888888888888888888888888。react() 使用 onlySystem，回调使用 SYSTEM.requestCallbackV_1_0。旧系统地址仍可用于 Cron 事件。Foundry 测试 vm cheatcode 不属于旧版 ReactVM。

不要提交助记词、私钥、钱包备份或 API 密钥。
