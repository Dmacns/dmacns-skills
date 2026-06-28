---
name: 1088-settle
description: Use when the user wants to check on-chain settlement status, view transaction history, or verify fund releases via their local 1088 node.
version: 1.0.0
author: 鼎脉数字 (DMACNS)
license: MIT
metadata:
  hermes:
    tags: [dmacns, 1088, settlement, chain]
    related_skills: [1088-setup, 1088-order]
---

# 1088 Settle — 链上结算查询

## Overview

教 Agent 通过本地 1088.exe 查询链上结算状态。包括：结算列表、结算汇总、单笔结算详情。
下单后资金进入链上托管（COMMIT），确认收货后自动释放（EXECUTE）。此 Skill 用于追踪这一过程。

## When to Use

- 用户问"我的钱到哪了"、"结算了吗"、"交易完成了吗"
- 用户想看交易历史和收入汇总
- 用户想确认某笔订单的链上状态

## Prerequisites

- 已通过 `1088-setup` 登录，`~/.dmacns/cookie.txt` 可用
- 已有完成的交易（至少一笔订单已完成或结算中）

## How to Run

### 结算列表

```python
import json, urllib.request, http.cookiejar, os

BASE = "http://localhost:18088"
COOKIE_FILE = os.path.expanduser("~/.dmacns/cookie.txt")

cj = http.cookiejar.MozillaCookieJar()
cj.load(COOKIE_FILE, ignore_discard=True)
opener = urllib.request.build_opener(urllib.request.HTTPCookieProcessor(cj))

def api(method, path, data=None):
    url = BASE + path
    body = json.dumps(data).encode() if data else None
    req = urllib.request.Request(url, data=body, method=method)
    req.add_header("Content-Type", "application/json")
    resp = opener.open(req)
    return json.loads(resp.read().decode())

# 结算列表（当前登录角色的 seller 视角）
settlements = api("GET", "/api/settlements")
if isinstance(settlements, list):
    print(f"结算记录: {len(settlements)} 笔\n")
    for s in settlements:
        print(f"  结算ID: {s.get('id')}")
        print(f"  订单ID: {s.get('order_id')}")
        print(f"  商品: {s.get('product_name')} x{s.get('quantity')}")
        print(f"  金额: ¥{s.get('amount')} | 运费: ¥{s.get('shipping_fee')} | 协议费: ¥{s.get('protocol_fee')}")
        print(f"  净收入: ¥{s.get('net_amount')} | 状态: {s.get('status')}")
        print(f"  创建: {s.get('created_at')} | 结算: {s.get('settled_at','待结算')}")
        print()
```

### 结算汇总

```python
summary = api("GET", "/api/settlements/summary")
print("结算汇总:")
print(f"  总订单: {summary.get('total_orders',0)}")
print(f"  待结算: {summary.get('pending_orders',0)}")
print(f"  已结算: {summary.get('settled_orders',0)}")
print(f"  总金额: ¥{summary.get('total_amount',0)}")
print(f"  总运费: ¥{summary.get('total_shipping',0)}")
print(f"  总协议费: ¥{summary.get('total_fee',0)}")
print(f"  净收入: ¥{summary.get('total_net',0)}")
```

### 结算字段说明

| 字段 | 说明 |
|------|------|
| `amount` | 订单总金额（商品价 × 数量） |
| `shipping_fee` | 运费 |
| `protocol_fee` | 协议服务费（通道费，0.5%） |
| `net_amount` | 净收入（amount - fee，卖家实收） |
| `status` | `pending`（待结算）/ `settled`（已结算） |
| `created_at` | 订单创建时间 |
| `settled_at` | 资金释放时间（null = 还未释放） |

## 链上结算流程

```
买家下单 → COMMIT（资金锁定到链上托管合约）
    ↓
卖家发货 → 买家确认收货 → EXECUTE（资金释放到卖家）
    ↓
settlement 状态: pending → settled
settled_at: null → 具体时间戳
```

**注意**：本地 1088 默认使用 mock 链上桥接，不真正上链。生产环境需配置 Conflux eSpace RPC。

## Common Pitfalls

1. **结算视角是卖家视角** — `/api/settlements` 返回的是当前登录用户作为卖家的结算记录。买家看自己的结算需要切换买家角色或查看订单列表。
2. **mock 模式** — 本地测试环境链上操作是模拟的，`escrow_id` 虽然存在但不是真实的链上交易哈希。
3. **协议费计算** — `protocol_fee = amount * 0.5%`，这是 DMACNS 通道费，在确认收货时自动扣除。

## Verification Checklist

- [ ] 结算列表正常返回（至少显示已创建的订单结算记录）
- [ ] 结算汇总中的数字与订单数一致
- [ ] `net_amount = amount - shipping_fee - protocol_fee`（近似）
