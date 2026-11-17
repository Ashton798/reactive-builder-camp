# D1 用例拆解卡：链上大额测试转账自动记录

## 一句话介绍

当 Ethereum Sepolia 测试网上指定代币合约发生达到阈值的转账时，Reactive Lasna 负责发现并判断，再请求 Base Sepolia 上的记录合约写下一条记录。

这是基于已完成的数字事件监听 Demo 提出的跨链扩展方案。下文的 Sepolia 转账监听和 Base Sepolia 回调尚未部署或测试。

## 1. 监听哪条链、哪个合约、什么事件？

源链是 Ethereum Sepolia 测试网。监听我们之后部署的测试 ERC-20 代币合约（具体地址部署后填写）的 Transfer(address indexed from, address indexed to, uint256 value) 转账事件。订阅指定源链、代币地址和事件主题，发送人和接收人使用通配符。

## 2. 满足什么条件触发？

Reactive 合约检查来源、主题、事件格式和转账数量；数量达到预设阈值，比如 100 个测试代币时，请求记录。实现时按代币 decimals 换算最小单位，并设置处理次数上限。例如转账 50 不记录，转账 120 才请求记录。是否排除铸造、销毁和重复事件，实施前要明确并测试。

## 3. 在哪条链做什么动作？

目标链是 Base Sepolia 测试网。Lasna 上的 Reactive 合约通过 requestCallbackV_1_0() 请求跨链回调；目标链回调代理调用记录合约，保存转账人、接收人、数量和事件唯一标识，并发出记录事件。接收合约须验证调用来自官方回调代理和授权的 Reactive 合约，准备目标链原生测试币支付回调费用，并处理重复记录和失败重试。这是计划，并非已经实现。

## 4. 不使用 Reactive 会怎样？好处和代价是什么？

可以自行运行服务器或机器人，持续读取 Sepolia 转账日志，满足条件后管理密钥并向 Base Sepolia 发送交易；也可以人工查看、手动提交，需要自己维护服务、交易和重试。

使用 Reactive 后，订阅和判断写进合约，由网络监听并传递事件，通过回调请求目标链动作。代价是要支付 Reactive 执行费和目标链回调费，管理资金和欠费，留意网络支持、延迟、重复事件、执行失败及回调身份校验；自动化不保证即时或百分百成功。

## 简图

源链 Ethereum Sepolia：测试 ERC-20 发出 Transfer
    ↓
Reactive Lasna：监听 → 判断来源、格式及数量是否 ≥ 100
    ↓ 满足条件：requestCallbackV_1_0
目标链 Base Sepolia：回调代理 → 验证身份 → 记录转账信息

## 已完成实践和计划的区别

已经完成的是另一条较简单的测试：Lasna 测试合约发出数字 15，Lasna Reactive 合约判断 15 ≥ 10 后自动记录。链上查询 count = 1、lastValue = 15；它证明了事件监听和自动处理，没有证明本卡描述的跨链回调。

[发出事件的交易](https://lasna.reactscan.net/tx/0x4d19281d68932ed55bea024fac45154e2daf5bbdacca50f8b9dbd8ab27534261)

[Reactive 自动处理的交易](https://lasna.reactscan.net/tx/0x756ad725d1000dc71a5c8d4ebff4a48f74c601d7c078acb79464a0db3bc7bff2)

参考：[Reactive Contracts](https://dev.reactive.network/reactive-contracts)、[Callback Contracts](https://dev.reactive.network/callback-contracts)、[Economy](https://dev.reactive.network/economy)。
