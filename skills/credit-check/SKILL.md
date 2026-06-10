---
name: credit-check
description: Use when the user wants to evaluate a supplier's credibility, verify business registration, check company operational status, or assess counterparty risk before a transaction. Searches public business registries and trade records.
version: 1.0.0
author: 鼎脉数字 (Dingmai Digital)
license: MIT
metadata:
  hermes:
    tags: [business, procurement, credit, risk-management]
    related_skills: [product-search, multi-sourcing, price-lock]
---

# Credit Check — 供应商信用评估

## 触发条件

用户表达以下意图时加载此技能：
- "查一下这家公司"
- "这卖家靠谱吗"
- "帮我看看他们的信用"
- "这个供应商有没有风险"

## 执行流程

### 1. 确认查询目标

明确要查询的主体：
- **公司全称**（最优）
- **统一社会信用代码**（最准）
- **店铺名称**（次选，需反查背后的公司）

### 2. 多渠道核实

#### 2.1 国家企业信用信息公示系统

使用 `web_search` 搜索：`site:gsxt.gov.cn <公司名>`

返回关键字段：
- 统一社会信用代码
- 法定代表人
- 注册资本
- 成立日期
- 经营范围
- 登记状态（存续/注销/吊销）
- 行政处罚/经营异常/严重违法记录

#### 2.2 天眼查 / 企查查 / 爱企查

使用 `web_search` 搜索：`<公司名> 天眼查`

补充信息：
- 司法风险（诉讼/被执行）
- 经营风险（行政处罚/环保处罚）
- 股东/实际控制人

#### 2.3 平台评价

如果供应商来自 1688/淘宝等平台，搜索店铺评价：
- 经营年限
- 交易等级
- 买家评价（尤其差评）
- 是否加入诚信保障

### 3. 输出评估报告

```
📋 <公司名> 信用评估

| 项目 | 内容 | 状态 |
|------|------|:--:|
| 经营状态 | 存续 | ✅ |
| 成立时间 | 2019年 | ✅ 5年+ |
| 注册资本 | ¥100万 | ✅ |
| 行政处罚 | 0条 | ✅ |
| 经营异常 | 0条 | ✅ |
| 司法诉讼 | — | — |

🎯 综合评分：🟢 低风险
```

### 4. 给建议

- 🟢 **低风险**：可放心交易
- 🟡 **中风险**：小额试单，分批付款
- 🔴 **高风险**：建议更换供应商，或要求预付款担保

## 注意事项

- 企业信息有公示延迟（通常 7-30 天），不能覆盖最近变更
- 无风险记录 ≠ 绝对安全，大额交易建议实地考察
- 新注册公司不一定有风险，但应谨慎

---

> 由鼎脉数字提供技术支持 | dmacns.com
