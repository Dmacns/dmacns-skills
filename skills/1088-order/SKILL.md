---
name: 1088-order
description: Use when the user wants to place an order, negotiate terms, or check order status via their local 1088 node. Covers the full order lifecycle: create, accept, ship, confirm receipt.
version: 1.0.0
author: 鼎脉数字 (DMACNS)
license: MIT
metadata:
  hermes:
    tags: [dmacns, 1088, order, trade]
    related_skills: [1088-setup, 1088-search, 1088-settle]
---

# 1088 Order — 下单与订单管理

## Overview

教 Agent 通过本地 1088.exe 完成交易全流程：下单、查订单、接受/发货/确认收货。

下单时 1088 自动执行链上 COMMIT（资金托管），卖家发货后买家确认 → 链上 EXECUTE（资金释放）。

## When to Use

- 用户说"帮我下单"、"买XX"、"采购XX"
- 用户想查看订单状态
- 用户作为卖家要接受订单或发货

## Prerequisites

- 已通过 `1088-setup` 登录，`~/.dmacns/cookie.txt` 可用
- `1088-search` 已完成搜索，手上有目标商品的 `sku` 和 `seller_did`

---

## 下单（买家操作）

### 前置：确认有收货地址

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

# 查现有地址
addrs = api("GET", "/api/address")
if addrs:
    addr = addrs[0]
    print(f"使用地址: {addr['id']} - {addr.get('province','')}{addr.get('city','')}{addr.get('detail','')}")
    ADDR_ID = addr["id"]
else:
    # 创建新地址
    r = api("POST", "/api/address", {
        "province": "山东省",      # ← 根据用户实际情况修改
        "city": "青岛市",
        "district": "市南区",
        "detail": "具体地址",
        "contact": "收货人姓名",
        "phone": "13800000000",
        "type": "shipping"
    })
    ADDR_ID = r["id"]
    print(f"新建地址: {ADDR_ID}")
```

### 下单

```python
# 从 1088-search 结果中获取的参数
SKU = "bf167478"            # ← 商品 SKU（来自搜索结果）
PRICE = 5.5                 # ← 单价
QTY = 20                    # ← 购买数量
SELLER_DID = "did:user:a_sale"  # ← 卖家 DID（来自搜索结果）

r = api("POST", "/api/orders", {
    "product_sku": SKU,
    "quantity": QTY,
    "unit_price": PRICE,
    "seller_did": SELLER_DID,
    "to_address_id": ADDR_ID,
    "buyer_note": ""         # 可选：给卖家的留言
})

order = r.get("order", {})
print(f"订单 ID: {order.get('id')}")
print(f"状态: {order.get('status')}")        # pending
print(f"金额: ¥{order.get('total_price')}")
print(f"链上托管: {order.get('escrow_id','无')}")
# → 下单成功即自动 COMMIT，资金进入链上托管
```

---

## 查订单

```python
orders = api("GET", "/api/orders")
for o in orders if isinstance(orders, list) else []:
    print(f"  {o.get('id')} | {o.get('product_name')} | ¥{o.get('total_price')} | {o.get('status')}")
```

**订单状态说明**：

| 状态 | 含义 | 下一步 |
|------|------|--------|
| `pending` | 待卖家接受 | 卖家操作：接受或拒绝 |
| `accepted` | 已接受，待发货 | 卖家操作：发货 |
| `shipped` | 已发货 | 买家操作：确认收货 |
| `completed` | 已完成 | 资金已释放到卖家 |
| `cancelled` | 已取消 | 资金退回买家 |
| `dispute` | 争议中 | 需仲裁 |

---

## 卖家操作（需要卖家角色登录）

如果用户身份是卖家（sales_manager），可以进行以下操作。

### 切换为卖家身份

用户需重新登录卖家账号（如 `a_sale / test123`），参考 `1088-setup` 的登录步骤。

### 接受订单

```python
ORDER_ID = "xxx"   # ← 订单 ID 从买家或订单列表获取
r = api("POST", "/api/orders/accept", {"order_id": ORDER_ID})
print(r)
```

### 发货

```python
r = api("POST", "/api/orders/ship", {
    "order_id": ORDER_ID,
    "carrier": "顺丰快递",
    "tracking_no": "SF1234567890"
})
print(r)
```

### 买家确认收货

```python
r = api("POST", "/api/orders/confirm", {"order_id": ORDER_ID})
print(r)
# → 确认收货触发链上 EXECUTE，资金释放给卖家
```

---

## 协商议价

```python
# 发起协商
r = api("POST", "/api/negotiations", {
    "order_id": "xxx",
    "proposed_price": 4.8,     # 还价
    "message": "量大能不能便宜点"
})
print(r)

# 查看协商记录
negs = api("GET", "/api/negotiations")
```

---

## Common Pitfalls

1. **角色权限** — 下单需要 `purchase_manager` 或 `operator`。如果用 `sales_manager` 登录会 403。切换角色用 `1088-setup` 换账号。
2. **seller_did 不能为空** — 搜索结果的 `seller_did` 字段必须传给下单 API。如果缺失，API 会返回"无法确定卖家身份"。
3. **地址必须存在** — 下单前确认有收货地址（`to_address_id`）。无地址则先创建。
4. **链上托管是 mock 模式** — 本地 1088 默认 mock 模式，不真正上链。生产环境需配置 Conflux eSpace RPC。

## Verification Checklist

- [ ] 下单成功，返回了 order id 和 `status: pending`
- [ ] 订单出现在 `GET /api/orders` 列表中
- [ ] 链上托管 ID（escrow_id）不为空
- [ ] 如果做卖家操作，确认已切换到卖家角色
