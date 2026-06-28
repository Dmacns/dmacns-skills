# DMACNS Skills

> 给你的 AI Agent 装上商业大脑：搜货源 → 比价格 → 查信用 → 下单 → 链上结算

## 这是什么

9 款 AI Agent 技能，让你的 Agent（Hermes、Claude Code、小艺、豆包等）学会 B2B 采购和链上交易。符合 **SKILL.md 开放标准**，即装即用。

## 技能体系

### 🍬 糖衣技能（零门槛，在线可用）

无需安装任何软件，Agent 加载即可用。适合初次接触的用户。

| # | 技能 | 做什么 |
|:--:|------|--------|
| 1 | **product-search** | 搜索供应商和产品 |
| 2 | **multi-sourcing** | 多方比价，自动对比 |
| 3 | **credit-check** | 查供应商信用和历史 |
| 4 | **price-lock** | 锁定报价，防止涨价 |
| 5 | **trade-escrow** | 交易资金托管 |

### 🖥️ 本地技能（需运行 1088.exe，P2P 直连）

Agent 直接操作本地 1088 节点，实现完整的 P2P 交易和链上结算。

| # | 技能 | 做什么 | 前置 |
|:--:|------|--------|------|
| 6 | **1088-setup** | 连接本地 1088 节点并登录 | 1088.exe 已启动 |
| 7 | **1088-search** | P2P 全网搜索商品 | 1088-setup |
| 8 | **1088-order** | 下单/协商/发货/确认收货 | 1088-setup + 1088-search |
| 9 | **1088-settle** | 链上结算查询与汇总 | 1088-setup + 已有订单 |

## 安装

### 糖衣技能（零前置）

```bash
# Hermes Agent
hermes skills install product-search
hermes skills install multi-sourcing
hermes skills install credit-check
hermes skills install price-lock
hermes skills install trade-escrow

# 或直接从 GitHub 安装
hermes skills install https://raw.githubusercontent.com/Dmacns/dmacns-skills/main/skills/product-search/SKILL.md
```

### 本地技能（需先安装 1088）

```bash
# 1. 下载并启动 1088.exe（从 https://dmacns.com 获取）
# 2. 安装 1088 Skill 群
hermes skills install 1088-setup
hermes skills install 1088-search
hermes skills install 1088-order
hermes skills install 1088-settle
```

### Claude Code / 其他 Agent

将 `skills/` 下的对应目录复制到平台的 skills 路径：

```bash
cp -r skills/1088-setup ~/.claude/skills/
```

支持所有兼容 SKILL.md 标准的平台（Cursor、Codex CLI、小艺、豆包等）。

## 使用示例

```
# 糖衣技能 — 零门槛
用户: 帮我找义乌的蓝牙耳机供应商
Agent: 加载 product-search → 返回结构化供应商列表

# 本地技能 — 完整交易
用户: 搜一下"烟台红富士苹果"
Agent: 加载 1088-search → P2P 全网搜索 → 返回报价

用户: 帮我下单买 20 斤
Agent: 加载 1088-order → 自动 COMMIT 链上托管 → 订单已创建

用户: 钱到了吗？
Agent: 加载 1088-settle → 结算查询 → 资金已释放
```

## 技能架构

```
                         dmacns-skills
                    ┌──────────────────┐
                    │   糖衣技能 ×5     │  ← 零门槛，获客入口
                    │  在线可用         │
                    ├──────────────────┤
                    │   本地技能 ×4     │  ← 需 1088.exe，完整交易
                    │   P2P + 链上结算  │
                    └──────────────────┘
                              │
                     DMACNS 商业网络
                  六原语 · P2P · e-CNY
```

## 开放标准

本仓库所有技能遵循 **SKILL.md 开放标准**（YAML frontmatter + Markdown body），兼容 Hermes、Claude Code、MCP、A2A 等主流 Agent 框架。

## 技术支持

> 由 [鼎脉数字 · DMACNS](https://dmacns.com) 提供技术支持

## 许可

MIT License
