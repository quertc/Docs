# 在 Morpheus 中构建基础：分阶段 AMM 部署

## 介绍

根据《Techno Capital Machine（TCM）》文档中概述的框架，最初的 90 天流动性引导阶段对于建立基础性流动性至关重要。在这个阶段，资本提供者的任务是将他们的 stETH 收益分配给 Morpheus，为生态系统的增长和稳定做出贡献。作为回报，资本提供者将获得每日 MOR 发行量的 24%。另外，每日 MOR 发行量的 4% 也被指定为保护基金，作为协议拥有的流动性（PoL），在初始化 Uniswap 自动做市商（AMM）池时发挥关键作用。

TCM 划分了五个关键资源类别：代码、资本、计算、社区和保护基金。在 90 天的流动性引导阶段之后，只有总发行量的 52% 将处于活跃状态。这个流通量包括代码提供者（24%）、资本提供者（24%）和保护基金（4%）MOR 分配的发行量。分配给社区（24%）和计算（24%）的发行量将在随后的阶段保持非流通状态。

## TL;DR

* AMM 的启动和公平价格发现对 Morpheus 至关重要
* 保护基金将其从 90 天流动性引导阶段分配的 4% MOR 发行量用于初始化 AMM 流动性池
* 到第 91 天，迄今为止发行的 MOR 仅有 52% 处于活跃状态
* AMM 启动阶段的策略将 Morpheus 获得的 52% 收益与保护基金的 MOR 配对
* Morpheus 获得的剩余 48% 收益分配给 Uniswap V3 集中流动性范围，从 0 调整到初始化比率（也称为价格），每周一次根据市场价格调整
* 资本提供者在 30 天的 AMM 启动阶段期间产生的收益周期性地增加到流动性中
* 在 30 天的 AMM 启动阶段之后，将进行一个过渡期（称为 AMM 平衡阶段），在此期间将调整集中的流动性支持范围（LSR）到完整范围的流动性位置
* 四个启动阶段：
1. 流动性引导阶段（Day 0 - 90）
2. AMM 启动阶段（Day 90 - 120）
3. AMM 平衡阶段（Day 120 - 220）
4. 长期稳态（Day 220+）

# 流动性引导阶段（第 0 天 - 第 90 天）

Morpheus 的公平启动于 2024 年 2 月 8 日 12:00 UTC，标志着流动性引导阶段的开始。该阶段的目标是通过 Techno Capital Machine（TCM）模型为 Morpheus 生成收益。这些概念在 GitHub 上有更详细的描述，但为了提供背景，以下是流动性引导阶段的简要描述：
* 资本提供者向 Morpheus 存入 stETH
* Morpheus 从 stETH 中获得收益（本金仍由资本提供者持有）
* 作为回报，资本提供者获得每日 MOR 发行量的 24%（每个存款人按比例获得其份额）。这些 MOR 发行量直到第 90 天才能领取。

Morpheus 从资本提供者获得的 stETH 收益是使 AMM 启动阶段开始的重要组成部分。

## AMM 启动阶段（第 90 天 - 第 120 天）

为了在这个初创阶段优化资源的部署，建议将 Morpheus 从资本提供者那里获得的收益的 52%，截止到第 88 天累积（注：第 88 天的使用是为了在 MOR 可领取和可转移之前创建池。这样可以给开发人员足够的时间在第 91 天启动之前准备好池），与同一时间段内保护基金获得的所有 MOR 配对。这种策略与白皮书和 TCM 中设定的长期流动性提供目标一致，尽管根据第 90 天实际流通供应量进行了修改。

用于管理剩余 48% 的 stETH（转换为 wETH）收益的策略包括将其分配到 Uniswap V3 集中流动性范围中，该范围从 0 到初始化比率，从而增强池中的 wETH 流动性并促进有效的市场活动。这个流动性支持范围（LSR）充当额外的流动性支持 - 如果 MOR 对 wETH 的价格下降，价格将移动到 LSR，并有助于减少价格影响。LSR 每周根据当前市场价格重新评估和调整一次。如果市场价格超过 LSR 的上限，上限将重新校准到新的价格水平，此时进行重新平衡。如果市场价格在 LSR 范围内，将不会进行调整，直到下次每周审查。这种机制设计为 30 天，之后，LSR 将过渡到包含完整范围（AMM 平衡阶段），增强整体市场稳定性和流动性。

这种方法确保了资源的有序和战略性部署，促进了健壮和流动的市场环境，与 TCM 和更广泛的 Morpheus 生态系统的长期目标一致。

### 利用 AMM 启动阶段生成的收益

为了支持公平的价格发现而不需要外部干预，Morpheus 将定期将 AMM 启动阶段的 30 天内生成的收益添加到流动性池中。
在 AMM 启动阶段结束后，根据 Morpheus 的白皮书，资本提供者智能合约被安排将收到的 stETH 收益的 50% 分配给 Morpheus 交换功能。该功能启动交换，从 AMM 购买 MOR 代币，将它们与剩余的 50% stETH（转换为 wETH）结合，并将其锁定到 AMM 作为协议所有的流动性。

## AMM 平衡阶段（第 120 天 - 第 220 天）

流动性支持范围的目标是在 AMM 启动阶段结束时扩展到完整范围位置（第 120 天），流动性支持可能出现两种情况：
1. 价格高于流动性支持范围，这意味着流动性支持范围包含 100% 的 wETH 和 0% 的 MOR
2. 价格在流动性支持范围内某处，这意味着平衡包含大于 0% 的 MOR

要从集中位置过渡到 50-50 的 wETH 和 MOR 平衡需要一些机制。因此，Morpheus 需要一些机制来实现这个比例。
#### 情景 1：如果在 AMM 启动阶段结束时流动性支持范围超过 50% 的 wETH，则将执行以下策略：
多余的 wETH 将在接下来的 100 天内逐步分配到 Morpheus 从资本提供者那里获得的每日收益流中，并且其中的一半随后会转换为 MOR 并添加到完整范围的流动性中。这种方法确保了过剩 wETH 的稳定和战略性的部署，增加了常规收益流，而不会压倒市场或对流动性位置产生不利影响。

#### 情景 2：如果流动性支持范围主要是 MOR（超过 50%），则策略将调整如下：
额外的 MOR 持有将由 Morpheus 保留用于未来与每日收益匹配，从而有效地消除了将一半的 stETH 收益转换为 MOR 的需求。这将每日持续进行，直到多余的 MOR 耗尽。

## 长期稳定状态（Day 220+）

在此阶段，协议将恢复到白皮书中概述的程序。资本提供者智能合约将提供产生的stETH收益的50％给Morpheus交换功能。该交换功能从自动市场做市商（AMM）购买MOR代币，然后将其与剩余的50％stETH（转换为wETH）配对，并将其作为PoL添加到流动性池中。

## 附录

### 附录 1：AMM启动 Day.88 可视化模拟
#### 请注意，提供的所有数字都是假设的，仅用于展示目的。这些数字不应被视为预测，也不应给出任何有关预期价格的指示。有关价格的所有讨论和预测，请参阅社区提供的正确的Discord频道和资源。

* 保护基金MOR：50,309.95
* 用于演示的88天Morpheus收益：500 ETH
* 用于启动的收益的52%：260 ETH
* 用于流动性支持范围的收益的48%：240 ETH

![1](https://github.com/generativeone/Images/blob/main/1.png)

### 附录 2：范围示例：

* 初始比率：0.00516796 ETH / MOR
* 流动性支持资金：240 ETH
* 流动性支持范围：0.0000000 至 0.00516796

如果市场价格在启动后一周（或上一个重新平衡周期）的12:00 pm UTC时落入流动性支持范围，则不会调整流动性支持范围。

初始比率：0.00516796 ETH / MOR

![range1](https://github.com/generativeone/Images/blob/main/range1-.png)

但是，如果市场价格上涨，则会调整流动性支持范围。例如，如果市场价格上涨到0.01033592 ETH / MOR，则新的支持范围将如下所示：

市场价格：0.01033592 ETH / MOR

![range2](https://github.com/generativeone/Images/blob/main/range2.png)

### 附录 3：AMM 平衡阶段示例

![2](https://github.com/generativeone/Images/blob/main/2-.png)

** 所有价格以MOR每wETH表示

** 步骤按日发生以简化数字表达。实际交易将按周频率进行。

### 附录 4：各阶段和关键事件的时间表

![3](https://github.com/generativeone/Images/blob/main/3.png)

### 附录 5：各个阶段用户可执行的操作

![4](https://github.com/generativeone/Images/blob/main/4.png)

### 附录 6：术语表

* AMM（Automated Market Maker）：依赖数学公式定价资产的去中心化交易所协议。与传统交易所使用订单簿不同，资产根据定价算法定价。
* Bootstrapping Period：协议启动初期的指定时期，旨在建立基础流动性，并通过奖励贡献者鼓励早期参与。
* 资本提供者：供应资本（在本例中为stETH收益）给协议的生态系统参与者。
* 代码提供者：为协议的基础代码和技术的开发和维护做出贡献的个人或实体。
* 计算：指协议运行所需的计算资源。
* 社区：创建接口、为Morpheus软件和代理提供贡献的建设者的集体。
* 集中流动性范围（LSR）：Uniswap V3流动性池内的特定集中价格范围。
* ETH（Ethereum）：以太坊区块链的原生加密货币，用于支付交易费用和计算服务。
* 全范围流动性：在自动做市商（AMM）中的一种策略，其中资本分配到交易对的整个价格范围，确保在所有价格水平都提供流动性。
* 启动比率：用于确定初始价格并启动AMM池的MOR与ETH的比率。
* MOR：Morpheus生态系统的原生代币。
* PoL（Protocol Owned Liquidity）：由协议本身拥有和控制的流动性，而不是由个人流动性提供者拥有。这有助于确保协议的长期可持续性和独立性。
* 保护基金：用于维护协议稳定性并初始化AMM流动性池的资产储备。
* stETH收益：在以太坊区块链上通过Lido协议质押ETH而产生的收益。
* TCM（Techno Capital Machine）：概述Morpheus生态系统的战略愿景和运营机制的基础文件或框架。
* Uniswap V3：Uniswap协议的第三次迭代，引入了集中流动性和多个费用等级。
* wETH（wrapped Ethereum）：代表基础资产的“包装”版本的令牌。这种包装版本被创建以允许附加功能，如存在于非本机链上。
