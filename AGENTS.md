# SunDAO 项目指南（AI Agent 阅读版）

> 本文档面向 AI 编程助手。阅读本文档前，默认你对 SunDAO 项目一无所知。
>
> **关键前提**：当前仓库（`SunDAO/`）是**总项目协调仓库**，本身**不含任何源代码**。所有代码位于同级目录的独立子项目仓库中。

---

## 一、项目概览

**深道（SunDAO）** 是下一代去中心化 AI 认知基础设施，为 AI Agent 提供可持久化、可溯源、可推理的语义数据基座。

核心设计哲学：
- **一个原语**：Being（实体）。一切数据皆实体，一切实体皆有身份、有状态、有历史、有关系。
- **一种语义**：记忆即不可变追加。实体不产生 UPDATE，只产生新版本。存储层本身就是完整历史。
- **一次查询**：跨维度统一。图引擎、列引擎、向量索引、全文索引在同一查询计划内协作。

核心三元组：**Being（实体）+ Type（类型）+ Relation（关系）**

---

## 二、仓库结构（重要）

当前仓库（`SunDAO/`）**仅包含文档**，不存储任何源代码。

```
SunDAO/                      # ← 当前仓库（本文档所在位置）
├── README.md                # 项目总览入口
├── .gitignore
├── AGENTS.md                # ← 本文档
├── design/                  # 设计草稿（空，待使用）
└── docs/                    # 总项目文档（中文）
    ├── 01-项目介绍.md
    ├── 02-架构总览.md
    ├── 03-子项目划分.md
    ├── 04-路线图.md
    ├── 05-开发指南.md       # 最重要的参考文档：构建、编码规范、测试、Git工作流
    └── 06-商业计划书.md
```

**实际的源代码仓库位于同级目录**，需要在文件系统中向上查找：

```
~/Workspace/
├── SunDAO/                  # ← 当前仓库（文档协调）
├── SunDaoQL/                # ← Rust Cargo Workspace，核心存储引擎（16个crate）
├── SunDaoQLServer/          # ← Rust Cargo Workspace，WebSocket 服务层
├── SunDaoCode/             # ← Node.js 应用，Web 前端
├── SunDaoAgent/             # ← Rust Crate，AI Agent 框架
├── SunDaoLLM/               # ← Rust Cargo Workspace，LLM 推理引擎
└── SunDaoApp/               # ← 规划中，跨平台客户端（尚未创建）
```

**注意**：这些子项目**不是**本仓库的 git submodule，而是独立的同级目录仓库。修改代码时必须切换到对应的子项目目录。

---

## 三、技术栈

| 层级 | 子项目 | 技术栈 | 类型 |
|------|--------|--------|------|
| 客户端 | SunDaoCode | Node.js + Express + 原生 HTML/CSS/JS | 应用 |
| 服务层 | SunDaoQLServer | Rust 1.85+ (2024 edition), tokio, tokio-tungstenite | 可执行服务 |
| 存储层 | SunDaoQL | Rust 1.85+ (2024 edition), Cargo Workspace (16 crates) | 库 |
| AI 层 | SunDaoAgent | Rust 1.85+ | 库 |
| AI 层 | SunDaoLLM | Rust 1.85+, candle-core, tokenizers | 库 |
| 客户端 | SunDaoApp | 待定（Electron / Neutralino.js / Tauri 选型中） | 应用 |

### 关键外部依赖选型
- **序列化**：postcard（二进制零拷贝）
- **全文索引**：tantivy + jieba 中文分词
- **位图索引**：roaring
- **时间索引**：redb
- **脚本引擎**：Rhai
- **监控**：Prometheus + tracing + OpenTelemetry

---

## 四、构建与测试命令

### 4.1 环境要求

| 组件 | 最低版本 |
|------|----------|
| Rust | 1.85+ |
| Node.js | 20+ |
| Git | 2.40+ |

初始化 Rust 工具链：
```bash
rustup update stable
rustup component add clippy rustfmt
cargo install cargo-nextest cargo-deny cargo-outdated
```

### 4.2 核心存储引擎（SunDaoQL）

```bash
cd ../SunDaoQL

# 调试构建
cargo build --workspace

# 发布构建
cargo build --workspace --release

# 运行全部测试
cargo test --workspace

# 运行特定包的测试
cargo test --package sundaoql-graph

# 运行特定测试
cargo test --package sundaoql-graph test_graph_engine_alloc_and_read_node

# 基准测试
cargo bench --bench xy_erp_stress
```

### 4.3 服务层（SunDaoQLServer）

```bash
cd ../SunDaoQLServer

cargo build --release
cargo run --bin sundaoql-server -- --config sundaoqlserver.toml
```

### 4.4 Web 前端（SunDaoCode）

```bash
cd ../SunDaoCode

npm install
node server.js    # 默认监听 http://localhost:3000
```

### 4.5 Agent 框架（SunDaoAgent）

```bash
cd ../SunDaoAgent
cargo build
cargo test
```

### 4.6 LLM 引擎（SunDaoLLM）

```bash
cd ../SunDaoLLM

# 仅 trait 定义（最小依赖）
cargo build

# 包含远程 API 客户端
cargo build --features remote

# 包含完整本地推理（需要模型文件）
cargo build --features inference
```

---

## 五、代码风格规范

### 5.1 Rust 命名规范

| 类型 | 规范 | 示例 |
|------|------|------|
| struct / enum / trait | 大驼峰 | `GraphEngineImpl`, `WalOp` |
| 函数 / 方法 | 小写下划线 | `write_node_at`, `traverse_bfs` |
| 变量 | 小写下划线 | `node_store`, `edge_offset` |
| 常量 | 全大写下划线 | `NODE_RECORD_SIZE`, `REL_HAS_PARENT` |
| 模块 | 小写下划线 | `graph_engine`, `wal_writer` |
| 宏 | 小写下划线 | `debug_graph_op!` |

### 5.2 Rust 文档注释规范

- 使用 `///` 为公共 API 编写文档注释。
- 文档注释使用**中文**（与项目整体语言一致）。
- 代码块中使用 `[`Name`]` 语法链接到相关类型。
- 复杂类型需包含 `# 示例` 章节。

```rust
/// 图引擎的具体实现。
///
/// 封装 [`NodeStore`]、[`EdgeStore`] 和 [`IndexEngine`]，
/// 实现 [`GraphEngine`] trait。
///
/// # 示例
///
/// ```ignore
/// let engine = GraphEngineImpl::new(node_store, edge_store, index_engine, metrics, 0.85)?;
/// let node = engine.read_node(offset)?;
/// ```
pub struct GraphEngineImpl { ... }
```

### 5.3 错误处理规范

- 使用 `thiserror` 定义错误类型。
- 错误消息使用中文或英文（保持一致即可）。
- 使用 `?` 传播错误。

```rust
#[derive(Debug, thiserror::Error)]
pub enum DaoError {
    #[error("CRC mismatch: expected {expected}, actual {actual}")]
    CrcMismatch { expected: u32, actual: u32 },

    #[error("record corrupted at offset {offset}")]
    RecordCorrupted { offset: u64 },
}
```

### 5.4 JavaScript 规范（SunDaoCode）

- 使用 ES2022 语法。
- 模块使用 ES Module (`import` / `export`)。
- 变量使用 `const` / `let`，禁止 `var`。
- 异步使用 `async` / `await`。

---

## 六、测试策略

### 6.1 Rust 测试层级

| 层级 | 类型 | 说明 | 命令示例 |
|------|------|------|----------|
| 单元测试 | `#[test]` | 单个函数/模块，无 I/O | `cargo test --package sundaoql-graph` |
| 集成测试 | `tests/*.rs` | 跨模块协作，使用 tempfile | `cargo test --test dsl_erp_integration` |
| 基准测试 | `benches/*.rs` | Criterion.rs，性能回归 | `cargo bench --bench xy_erp_stress` |
| E2E 测试 | `tests/*_e2e.rs` | 完整服务链路 | `cargo test --test replication_e2e` |

### 6.2 测试规范

- 使用 `tempfile::TempDir` 创建隔离测试目录。
- 使用 `sample_being_core()` 等 fixture 函数生成测试数据。
- 测试完成后自动清理。

```rust
#[test]
fn test_graph_engine_alloc_and_read_node() {
    let dir = TempDir::new().unwrap();
    let mut engine = create_engine(dir.path());

    let offset = engine.alloc_node().unwrap();
    let core = sample_being_core(0);
    engine.write_node_at(offset, &core).unwrap();

    let read_back = engine.read_node(offset).unwrap();
    assert_eq!(read_back.id, core.id);
    assert_eq!(read_back.def, core.def);
}
```

---

## 七、Git 工作流

### 7.1 分支模型

- `main`：稳定分支
- `feature/xxx`：功能分支
- `release/vx.y`：发布分支
- `hotfix/xxx`：热修复分支

### 7.2 提交规范

格式：
```
<type>: <subject>

<body>

<footer>
```

**类型**：

| 类型 | 说明 |
|------|------|
| `feat` | 新功能 |
| `fix` | 修复 bug |
| `docs` | 文档更新 |
| `style` | 代码格式（不影响功能） |
| `refactor` | 重构 |
| `perf` | 性能优化 |
| `test` | 测试相关 |
| `chore` | 构建/工具链 |

示例：
```
feat: 实现Column引擎ProjectedLayer物化列自动创建

- 检测热点字段（出现频率>10%）
- 自动创建.col列存文件和.idx跳数索引
- 后台异步线程执行，不阻塞写入路径

Closes #42
```

---

## 八、调试与诊断

### 8.1 日志级别

```bash
# 开发调试
RUST_LOG=debug cargo run

# 查看特定模块
RUST_LOG=sundaoql_graph=trace cargo test

# 生产环境
RUST_LOG=info cargo run --release
```

### 8.2 常用诊断命令

```bash
# 检查依赖树
cargo tree --package sundaoql-core

# 检查重复依赖
cargo tree --duplicates

# 安全检查
cargo audit

# 检查未使用依赖
cargo machete

# 格式化代码
cargo fmt

# 静态检查（零警告）
cargo clippy --workspace -- -D warnings
```

### 8.3 性能分析

```bash
# Criterion 基准测试
cargo bench --bench xy_erp_stress

# 火焰图（需 cargo-flamegraph）
cargo flamegraph --bench xy_erp_stress
```

---

## 九、发布检查清单

- [ ] 所有测试通过 (`cargo test --workspace`)
- [ ] 静态检查无警告 (`cargo clippy`)
- [ ] 格式化通过 (`cargo fmt --check`)
- [ ] 基准测试无性能回归 (`cargo bench`)
- [ ] 文档已更新
- [ ] CHANGELOG 已更新
- [ ] 版本号已更新 (`Cargo.toml`)

---

## 十、安全注意事项

- **国密算法**：存储层使用 SM2/SM3/SM4 国密算法，WAL 和 RawLayer 默认加密。
- **传输加密**：WebSocket 通道使用 SM4-CTR 或 TLS 1.3。
- **认证机制**：ProKey 渐进式密钥认证（Ed25519 签名 + Nonce 防重放）。
- **权限控制**：CBAC（基于角色的访问控制）+ ACL（字段级权限）+ 多租户隔离。
- **审计**：WAL 即审计日志，不可变追加，天然满足审计需求。
- **闭源软件**：本项目为闭源软件，未经许可不得复制、修改、分发。

---

## 十一、子项目快速索引

| 子项目 | 路径（相对于当前仓库） | 类型 | 状态 |
|--------|------------------------|------|------|
| SunDaoQL | `../SunDaoQL/` | Rust Cargo Workspace（库） | 开发中 |
| SunDaoQLServer | `../SunDaoQLServer/` | Rust Cargo Workspace（服务） | 开发中 |
| SunDaoCode | `../SunDaoCode/` | Node.js 应用 | 开发中 |
| SunDaoAgent | `../SunDaoAgent/` | Rust Crate（库） | 开发中 |
| SunDaoLLM | `../SunDaoLLM/` | Rust Cargo Workspace（库） | 开发中 |
| SunDaoApp | `../SunDaoApp/` | 待定 | 规划中 |

---

## 十二、给 AI Agent 的特别提醒

1. **不要在本仓库中写代码**。当前仓库只有 Markdown 文档。写代码前务必 `cd` 到正确的子项目目录（如 `../SunDaoQL/`）。
2. **文档语言是中文**。所有代码注释、文档、提交信息均使用中文。
3. **单方向依赖**：上层依赖下层，禁止循环依赖。SunDaoQL 不能依赖 SunDaoAgent 或 SunDaoQLServer。
4. **SunDaoQL 是纯库**：没有 `main.rs`，不监听端口。网络入口必须在 SunDaoQLServer。
5. **当前阶段是 P4（基座打通）**：约 40% 完成度。大量功能仍在规划中，修改前请对照 `docs/04-路线图.md` 确认相关模块是否已启动。
6. **环境变量**：开发时建议设置 `RUST_LOG=info`。

---

*本文档基于 SunDAO 项目实际内容生成。*
*文档日期: 2026-05-25*
