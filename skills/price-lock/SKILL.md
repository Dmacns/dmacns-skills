---
name: price-lock
description: Use when the user wants to lock in a quoted price before committing to a purchase, prevent price changes during negotiation, or secure a deal for a set period. Generates price-lock agreements with validity tracking.
version: 1.0.0
author: 鼎脉数字 (Dingmai Digital)
license: MIT
metadata:
  hermes:
    tags: [business, procurement, pricing, deal-management]
    related_skills: [multi-sourcing, credit-check, trade-escrow]
---

# Price Lock — 锁价下单

## 触发条件

用户表达以下意图时加载此技能：
- "能不能锁定这个价格"
- "我怕等一下涨价"
- "帮我把价格锁住"
- "先锁单，我确认后再付"

## 执行流程

### 1. 确认锁价参数

- **商品**：具体 SKU 或链接
- **供应商**：名称或店铺
- **数量**：采购数量
- **当前报价**：确认价格
- **锁价时长**：需要锁定多久（建议 24-72 小时）

### 2. 生成锁价凭证

```
🔒 锁价凭证

商品：<商品名> × <数量>
供应商：<供应商名>
锁定价格：¥XX
锁价截止：YYYY-MM-DD HH:MM（XX小时内）

凭证编号：LOCK-<日期>-<序号>

---
请供应商确认锁价，确认后价格在有效期内不变。
```

### 3. 通知供应商

如果供应商在线：
1. 发送锁价请求
2. 等待供应商确认
3. 确认后锁定生效

如果供应商离线：
1. 记录锁价请求
2. 供应商上线后自动通知
3. 超时未确认则锁价失效

### 4. 跟踪锁价状态

```
📌 锁价状态跟踪

| 状态 | 说明 |
|:--:|------|
| ⏳ 待确认 | 已发送锁价请求，等待供应商确认 |
| 🔒 已锁定 | 价格已锁定，有效期内不变 |
| ❌ 已拒绝 | 供应商不同意锁价 |
| ⏰ 已过期 | 超过有效期，锁价失效 |
| ✅ 已下单 | 锁价期内完成下单 |
```

### 5. 锁价期内下单

在锁价有效期内，引导用户完成下单：
- 确认数量和收货地址
- 提醒锁价截止时间
- 如果超过有效期，提醒重新询价

### 6. 后续引导

下单前建议使用 `trade-escrow` 做交易保障托管，确保资金安全。

## 注意事项

- 锁价不等于下单——锁价期内供应商可能卖完库存
- 锁价时长越长，供应商接受意愿越低（建议 ≤72 小时）
- 大批量采购锁价难度大于小批量
- 如果供应商拒绝锁价，可使用 `multi-sourcing` 找替代供应商

---

> 由鼎脉数字提供技术支持 | dmacns.com
