# 深道（SunDAO）

> **去中心化AI认知基础设施**
>
> "道生一，一生二，二生三，三生万物"
>
> 以本体论为先、多模态融合、统一负载的数据平台。

---

## 🌟 项目定位

**深道（SunDAO）** 是下一代去中心化AI认知基础设施。它不是一个数据库，而是AI与世界的"认知接口"——为AI Agent提供可持久化、可溯源、可推理的语义数据基座。

核心三元组：**Being（实体）+ Type（类型）+ Relation（关系）**。一切数据皆实体，一切实体皆有身份、有状态、有历史、有关系。

---

## 📦 子项目全景

SunDAO 由以下六个子项目组成，各负其责，协同构成完整生态：

| 子项目 | 定位 | 类型 | 状态 |
|--------|------|------|------|
| **SunDaoQL** | 核心存储引擎。图引擎+列引擎+向量索引+全文索引+WAL+加密，单一Rust库，零外部依赖 | Rust库 | 开发中 |
| **SunDaoQLServer** | 服务层。DSL/Meta WebSocket服务，为客户端提供查询入口 | Rust服务 | 开发中 |
| **SunDaoQLWeb** | Web前端。基于Node.js的WebSocket客户端，连接SunDaoQLServer | Node.js应用 | 开发中 |
| **SunDaoAgent** | AI Agent框架。Being-Native Agent，MCP工具集成，多Agent协作，自我进化 | Rust库 | 开发中 |
| **SunDaoLLM** | LLM推理引擎。本地DeepSeek V4 Flash推理，KV Cache持久化，远程API客户端 | Rust库 | 开发中 |
| **SunDaoApp** | 跨平台客户端。Tauri v2 + Vue 3 动态UI渲染引擎，基于def/type/relation语义组装交互界面，覆盖PC/手机/平板/Telegram/微信 | Tauri+Vue应用 | 设计完成

```
┌─────────────────────────────────────────────────────────────┐
│                        客户端层                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│  │ SunDaoApp│  │SunDaoQLWeb│  │  MCP     │  │  CLI     │    │
│  │(PC/手机) │  │ (Web)    │  │ (Agent)  │  │ (脚本)   │    │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘    │
└───────┼─────────────┼─────────────┼─────────────┼──────────┘
        │             │             │             │
        └─────────────┴──────┬──────┴─────────────┘
                             │ WebSocket / DSL
                             ▼
┌─────────────────────────────────────────────────────────────┐
│                    SunDaoQLServer                            │
│              DSL解析 · Meta服务 · 查询路由                    │
└────────────────────────┬────────────────────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   SunDaoQL   │  │ SunDaoAgent  │  │  SunDaoLLM   │
│  核心存储引擎 │  │ AI Agent框架 │  │ LLM推理引擎  │
│ 图·列·向量·全文│  │ MCP·记忆·进化 │  │ 本地·远程   │
└──────────────┘  └──────────────┘  └──────────────┘
```

---

## 🧠 核心设计理念

### 一个原语：Being（实体）

SunDAO 中的一切都是实体。客户是实体，订单是实体，「属于」「创建」「相似」也是实体（关系实体）。

每个实体自带 26 个通用业务字段：身份、分类、状态、版本、审计时间、操作人、租户、权限组、搜索编码……**你不再需要为每个表重复设计这些字段。**

### 一种语义：记忆（不可变追加）

实体不会"被修改"——只会"产生新版本"。旧版本仍在，只是不再是"当前"。

```rust
// 当前状态
sundaoql.query().being(order_id).fetch_one();

// 三个月前的状态
sundaoql.query().being(order_id).as_of("2025-01-15T00:00:00Z").fetch_one();
```

没有审计表，没有 CDC，没有日志管道。**存储层本身就是完整的历史。**

### 一次查询：跨维度统一

```rust
sundaoql.query()
    .being(current_project_id)
    .similar_to(TopK(10))           // 向量：最相似的项目
    .via_relation(RelationType::MANAGED_BY)  // 图：找到负责人
    .filter(|b| b.status == DELIVERED && b.updated_at > six_months_ago())  // 列：筛选条件
    .with_fulltext("供应链优化")    // 全文：描述匹配
    .execute();
```

图引擎、列引擎、向量索引、全文索引在同一查询计划内协作，**结果自动合并**。

---

## 🚀 快速开始

### 构建核心引擎

```bash
cd SunDaoQL
cargo build --workspace --release
cargo test --workspace
```

### 启动服务层

```bash
cd SunDaoQLServer
cargo run --bin sundaoql-server -- --config sundaoqlserver.toml
```

### 启动Web前端

```bash
cd SunDaoQLWeb
npm install
node server.js
```

---

## 📚 文档导航

| 文档 | 说明 |
|------|------|
| [docs/01-项目介绍.md](docs/01-项目介绍.md) | SunDAO项目总览与更名说明 |
| [docs/02-架构总览.md](docs/02-架构总览.md) | 系统架构与子项目关系 |
| [docs/03-子项目划分.md](docs/03-子项目划分.md) | 子项目职责边界与依赖关系 |
| [docs/04-路线图.md](docs/04-路线图.md) | 发展阶段与里程碑 |
| [docs/05-开发指南.md](docs/05-开发指南.md) | Workspace组织与开发规范 |
| [docs/06-商业计划书.md](docs/06-商业计划书.md) | 商业模式与市场分析 |

各子项目的详细设计文档位于各自仓库的 `docs/` 目录下。

---

## 🛡️ 版权

© SunDAO Team. 保留所有权利。

本项目为闭源软件，未经许可不得复制、修改、分发。
