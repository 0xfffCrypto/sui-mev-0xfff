# Sui MEV Bot


## Run 
Start the bot with your private key.

```bash
cargo run -r --bin arb start-bot -- --private-key {}
```

## Supports

- BlueMove
- FlowX
- Aftermath
- Cetus 
- Kriya
- Abex
- Navi
- Turbos
- Deepbook
- Shio

## Relay
If you have a validator, you can let the validator push mempool transactions to your relay, which then send to the bot.

```bash
cargo run -r --bin relay
```

## Workflow Diagram

```mermaid
graph TD
    subgraph DexIndexer [Dex Indexer Crate/Service]
        direction LR
        P1[Protocol: Cetus] --> IDXCOL{Collector}
        P2[Protocol: Kriya] --> IDXCOL
        P3[Protocol: Aftermath] --> IDXCOL
        P4[...] --> IDXCOL
        IDXCOL --> IDXDB[(Pool Data Store)]
    end

    subgraph ArbBot [Arbitrage Bot (bin/arb)]
        direction LR
        SRCH[DexSearcher] -->|Get Pools/Paths| IDXDB
        SRCH --> STRAT{Strategy/Worker}
        STRAT -->|Potential Path| TRDR{Trader}
        TRDR -->|Simulate Trade| SIM{Simulator}
        SIM -->|Simulation Result| TRDR
        TRDR -- Profit? -->|Build Tx| TXBLD[Transaction Builder]
        TXBLD -->|Submit Tx| SUBMIT{Submitter (e.g., Shio/Direct)}
    end

    subgraph ExternalData [External Data Sources]
        direction TB
        MEMPOOL[Mempool (Optional via Relay)] --> ArbBot
        BLOCKS[Sui Blockchain Events/State] --> DexIndexer
        BLOCKS --> SIM
    end

    ExternalData --> ArbBot
    DexIndexer --> ArbBot

    style DexIndexer fill:#f9f,stroke:#333,stroke-width:2px
    style ArbBot fill:#ccf,stroke:#333,stroke-width:2px
    style ExternalData fill:#dfd,stroke:#333,stroke-width:2px
```

## Adding a New DEX

Integrating a new Decentralized Exchange (DEX) involves the following steps:

1.  **Understand the DEX Contracts:**
    *   Identify the DEX's main Sui package ID.
    *   Find the specific Move module and function names for executing swaps.
    *   Determine the required function arguments (types and values) and type parameters.
    *   If using flash loans, find the functions and parameters for initiating and repaying loans.
    *   Identify any global configuration or state objects required for interaction.

2.  **Implement the `Dex` Trait (`bin/arb/src/defi/`):**
    *   Create a new file (e.g., `bin/arb/src/defi/new_dex.rs`).
    *   Define a struct (e.g., `NewDex`) to hold pool-specific state and object references.
    *   Implement the `Dex` trait (defined in `bin/arb/src/defi/mod.rs`) for your struct:
        *   **`extend_trade_tx` (Required):** Implement the core logic to add the correct `Command::move_call` to the `TradeCtx` for performing a swap on this DEX.
        *   **Flash Loan Methods (Optional):** If applicable, override `support_flashloan` to return `true` and implement `extend_flashloan_tx` and `extend_repay_tx`.
        *   **Metadata Methods (Required):** Implement `coin_in_type`, `coin_out_type`, `protocol`, `liquidity`, `object_id`.
        *   **Utility Methods (Required):** Implement `flip`, `is_a2b`, `swap_tx`.
    *   Add `mod new_dex;` to `bin/arb/src/defi/mod.rs`.
    *   Refer to existing implementations like `cetus.rs` or `kriya_amm.rs` for examples.

3.  **Integrate with the Indexer (`crates/dex-indexer/src/protocols/`):**
    *   Create a corresponding module (e.g., `crates/dex-indexer/src/protocols/new_dex.rs`).
    *   Implement logic to discover pools for the new DEX (e.g., by listening to events or querying contracts). This allows the bot to find arbitrage opportunities involving the new DEX.
    *   Register the new protocol module in `crates/dex-indexer/src/protocols/mod.rs`.
    *   Add necessary data files (like pool lists) in `crates/dex-indexer/data/` if needed.

4.  **Configuration (Optional):**
    *   If the new DEX requires specific configuration (e.g., unique API keys, feature flags), update the configuration logic (likely in `bin/arb/src/config.rs`).

5.  **Testing:**
    *   Add unit tests for your new DEX implementation, similar to those found in existing DEX modules (e.g., `bin/arb/src/defi/cetus.rs`'s test module).

## Development TODOs

-   **Extend DEX Support:**
    *   [ ] Add support for [New DEX Name 1]
    *   [ ] Add support for [New DEX Name 2]
    *   ... (add more DEXs as needed)
-   **Benchmarking:**
    *   [ ] Implement performance benchmarking for different arbitrage paths and DEX interactions.
    *   [ ] Track simulation times vs. actual execution times.
-   **Monitoring & Alerting:**
    *   [ ] Integrate real-time monitoring of bot health (e.g., connection status, error rates).
    *   [ ] Add alerts (e.g., via Telegram, Discord) for critical errors or successful high-profit trades.
-   **PNL Tracking Dashboard:**
    *   [ ] Develop a system to persistently track Profit and Loss (PNL) over time.
    *   [ ] Create a simple dashboard (web-based or terminal UI) to visualize PNL, trade history, and asset balances.
    *   [ ] Track gas costs accurately per trade/opportunity.
-   **Refactoring & Optimization:**
    *   [ ] Review and optimize hot paths in the arbitrage detection and execution logic.
    *   [ ] Improve error handling and resilience.

---

# Sui MEV 机器人 (中文版)

## 运行
使用你的私钥启动机器人。

```bash
cargo run -r --bin arb start-bot -- --private-key <你的私钥>
```

## 支持的 DEX

- BlueMove
- FlowX
- Aftermath
- Cetus
- Kriya
- Abex
- Navi
- Turbos
- Deepbook
- Shio

## 中继 (Relay)
如果你有验证节点，你可以让验证节点将内存池交易推送给你的中继服务，然后中继服务再发送给机器人。

```bash
cargo run -r --bin relay
```

## 工作流程图

```mermaid
graph TD
    subgraph DexIndexer [Dex Indexer Crate/Service]
        direction LR
        P1[Protocol: Cetus] --> IDXCOL{Collector}
        P2[Protocol: Kriya] --> IDXCOL
        P3[Protocol: Aftermath] --> IDXCOL
        P4[...] --> IDXCOL
        IDXCOL --> IDXDB[(Pool Data Store)]
    end

    subgraph ArbBot [Arbitrage Bot (bin/arb)]
        direction LR
        SRCH[DexSearcher] -->|Get Pools/Paths| IDXDB
        SRCH --> STRAT{Strategy/Worker}
        STRAT -->|Potential Path| TRDR{Trader}
        TRDR -->|Simulate Trade| SIM{Simulator}
        SIM -->|Simulation Result| TRDR
        TRDR -- Profit? -->|Build Tx| TXBLD[Transaction Builder]
        TXBLD -->|Submit Tx| SUBMIT{Submitter (e.g., Shio/Direct)}
    end

    subgraph ExternalData [External Data Sources]
        direction TB
        MEMPOOL[Mempool (Optional via Relay)] --> ArbBot
        BLOCKS[Sui Blockchain Events/State] --> DexIndexer
        BLOCKS --> SIM
    end

    ExternalData --> ArbBot
    DexIndexer --> ArbBot

    style DexIndexer fill:#f9f,stroke:#333,stroke-width:2px
    style ArbBot fill:#ccf,stroke:#333,stroke-width:2px
    style ExternalData fill:#dfd,stroke:#333,stroke-width:2px
```

## 添加新的 DEX

集成一个新的去中心化交易所 (DEX) 涉及以下步骤：

1.  **理解 DEX 合约:**
    *   识别 DEX 的主要 Sui Package ID。
    *   找到执行 Swap 的具体 Move 模块和函数名称。
    *   确定所需的函数参数（类型和值）以及类型参数。
    *   如果使用闪电贷，找到发起和偿还贷款的函数及参数。
    *   识别交互所需的任何全局配置或状态对象。

2.  **实现 `Dex` Trait (`bin/arb/src/defi/`):**
    *   创建一个新文件 (例如, `bin/arb/src/defi/new_dex.rs`)。
    *   定义一个结构体 (例如, `NewDex`) 来持有特定池的状态和对象引用。
    *   为你的结构体实现 `Dex` trait (定义在 `bin/arb/src/defi/mod.rs`):
        *   **`extend_trade_tx` (必需):** 实现核心逻辑，将执行此 DEX Swap 所需的正确 `Command::move_call` 添加到 `TradeCtx`。
        *   **闪电贷方法 (可选):** 如果适用，重写 `support_flashloan` 使其返回 `true`，并实现 `extend_flashloan_tx` 和 `extend_repay_tx`。
        *   **元数据方法 (必需):** 实现 `coin_in_type`, `coin_out_type`, `protocol`, `liquidity`, `object_id`。
        *   **工具方法 (必需):** 实现 `flip`, `is_a2b`, `swap_tx`。
    *   在 `bin/arb/src/defi/mod.rs` 中添加 `mod new_dex;`。
    *   参考现有的实现，如 `cetus.rs` 或 `kriya_amm.rs` 作为示例。

3.  **与 Indexer 集成 (`crates/dex-indexer/src/protocols/`):**
    *   创建一个对应的模块 (例如, `crates/dex-indexer/src/protocols/new_dex.rs`)。
    *   实现发现新 DEX 池子的逻辑 (例如, 通过监听事件或查询合约)。这使得机器人能够找到涉及新 DEX 的套利机会。
    *   在 `crates/dex-indexer/src/protocols/mod.rs` 中注册新的协议模块。
    *   如果需要，在 `crates/dex-indexer/data/` 中添加必要的数据文件 (例如池子列表)。

4.  **配置 (可选):**
    *   如果新的 DEX 需要特定配置 (例如, 唯一的 API 密钥, 功能开关)，更新配置逻辑 (可能在 `bin/arb/src/config.rs`)。

5.  **测试:**
    *   为你的新 DEX 实现添加单元测试，类似于现有 DEX 模块中的测试 (例如, `bin/arb/src/defi/cetus.rs` 的测试模块)。

## 开发待办事项 (TODOs)

-   **扩展 DEX 支持:**
    *   [ ] 添加对 [新 DEX 名称 1] 的支持
    *   [ ] 添加对 [新 DEX 名称 2] 的支持
    *   ... (根据需要添加更多 DEX)
-   **基准测试:**
    *   [ ] 实现针对不同套利路径和 DEX 交互的性能基准测试。
    *   [ ] 跟踪模拟时间与实际执行时间。
-   **监控与警报:**
    *   [ ] 集成机器人健康状况的实时监控 (例如, 连接状态, 错误率)。
    *   [ ] 为严重错误或成功的高利润交易添加警报 (例如, 通过 Telegram, Discord)。
-   **PNL 追踪仪表盘:**
    *   [ ] 开发一个系统来持久跟踪盈亏 (PNL)。
    *   [ ] 创建一个简单的仪表盘 (基于 Web 或终端 UI) 来可视化 PNL、交易历史和资产余额。
    *   [ ] 精确跟踪每笔交易/机会的 Gas 成本。
-   **重构与优化:**
    *   [ ] 审查和优化套利检测和执行逻辑中的热点路径。
    *   [ ] 改进错误处理和程序健壮性。
