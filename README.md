# AI 智能采购技能包

> 给你的 AI Agent 装上采购大脑：搜货源 → 比价格 → 查信用 → 锁订单 → 托管结算

## 这是什么

五款免费 AI Agent 技能，让你的 Agent（Claude Code、Cursor、Codex CLI、Hermes 等）学会 B2B 采购。符合 **SKILL.md 开放标准**，即装即用。

## 五款技能

| # | 技能 | 做什么 | 一句话 |
|:---:|------|--------|--------|
| 1 | **product-search** | 智能货源搜索 | "我要买什么 → Agent 帮你搜" |
| 2 | **multi-sourcing** | 多方比价采购 | "多家供应商自动对比" |
| 3 | **credit-check** | 供应商信用评估 | "下单前查对方信誉" |
| 4 | **price-lock** | 锁价下单 | "锁定价格，不怕涨价" |
| 5 | **trade-escrow** | 交易保障托管 | "付款先托管，收货再放款" |

## 安装

### Hermes Agent

```bash
hermes skills install product-search
hermes skills install multi-sourcing
hermes skills install credit-check
hermes skills install price-lock
hermes skills install trade-escrow
```

### Claude Code

将 `skills/` 下的目录复制到 `~/.claude/skills/`：

```bash
cp -r skills/product-search ~/.claude/skills/
```

### 其他平台

支持所有兼容 SKILL.md 标准的 AI Agent 平台（Cursor、Codex CLI、OpenCode 等），将对应目录放入平台的 skills 路径即可。

## 使用示例

```
用户: 帮我找义乌的蓝牙耳机供应商

Agent 自动加载 product-search → 搜索 1688/义乌购/拼多多 → 返回结构化供应商列表

用户: 帮我比一下这三家的价格

Agent 自动加载 multi-sourcing → 输出对比表

用户: 查一下这家公司的信用

Agent 自动加载 credit-check → 返回工商注册/经营年限/风险提示
```

## 开放标准

本仓库所有技能遵循 **SKILL.md 开放标准**（YAML frontmatter + Markdown body），由 Anthropic 发起，已被 40+ AI 产品采用。

## 技术支持

> 由 [鼎脉数字](https://dmacns.com) 提供技术支持

## 许可

MIT License
