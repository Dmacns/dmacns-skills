---
name: 1088-setup
description: Use when the user wants to connect their AI Agent to a local 1088 node. Verifies 1088.exe is running and logs in to obtain a session token.
version: 1.0.0
author: 鼎脉数字 (DMACNS)
license: MIT
metadata:
  hermes:
    tags: [dmacns, 1088, setup, authentication]
    related_skills: [1088-search, 1088-order, 1088-settle]
---

# 1088 Setup — 连接本地 1088 节点

## Overview

教 Agent 确认本地 1088.exe 已运行，并用 demo 账号登录获取 session cookie。
所有后续 Skill（search/order/settle）依赖此 Skill 建立的会话。

## When to Use

- 用户第一次使用 DMACNS 1088 Skill 系列时
- 需要切换 1088 账号时
- 登录状态丢失（cookie 过期）时

## Prerequisites

- **1088.exe 已在本地启动**（端口 18088）。如未启动，告知用户在 1088 目录双击 `1088.exe`
- Python 3 可用（`python` 命令）

## How to Run

### Step 1: 验证 1088 是否运行

```bash
python -c "import urllib.request; print(urllib.request.urlopen('http://localhost:18088/api/health').read().decode())"
```

预期输出包含 `"status":"ok"` 和 `"db":"ok"`。

如果连接失败（Connection Refused），告知用户启动 1088.exe 后重试。

### Step 2: 登录获取 session

1088 预置了 5 家公司各 3 个角色的 demo 账号（密码均为 `test123`）：

| 公司 | 采购经理 | 销售经理 | 操作员 |
|------|---------|---------|--------|
| A 烟台苹果贸易 | a_purc | a_sale | a_oper |
| B 青岛海鲜水产 | b_purc | b_sale | b_oper |
| C 济南机械制造 | c_purc | c_sale | c_oper |
| D 潍坊农产品 | d_purc | d_sale | d_oper |
| E 临沂物流仓储 | e_purc | e_sale | e_oper |

**用 Python 登录并保存 cookie**（中文安全，避免 MSYS curl 乱码）：

```python
import json, urllib.request, http.cookiejar, os

BASE = "http://localhost:18088"
COOKIE_FILE = os.path.expanduser("~/.dmacns/cookie.txt")

# 创建带 cookie 的 opener
cj = http.cookiejar.MozillaCookieJar(COOKIE_FILE)
opener = urllib.request.build_opener(urllib.request.HTTPCookieProcessor(cj))

def api(method, path, data=None):
    url = BASE + path
    body = json.dumps(data).encode() if data else None
    req = urllib.request.Request(url, data=body, method=method)
    req.add_header("Content-Type", "application/json")
    resp = opener.open(req)
    return json.loads(resp.read().decode())

# 登录（根据用户身份选择账号；默认用买家 a_purc）
USERNAME = "a_purc"   # ← 根据用户身份修改
PASSWORD = "test123"

r = api("POST", "/api/auth/login", {"username": USERNAME, "password": PASSWORD})
cj.save(COOKIE_FILE)
print(r)
# → {"ok": true, "username": "a_purc", "did": "did:user:a_purc", "company_name": "烟台苹果贸易", "role": "purchase_manager"}
```

**关键点**：
- `MozillaCookieJar.save()` 保存 cookie 到 `~/.dmacns/cookie.txt`，后续 Skill 复用
- 登录返回 `did` 和 `role`——记下来，下单/协商时需要

### Step 3: 验证登录状态

```bash
python -c "
import json, urllib.request, http.cookiejar
cj = http.cookiejar.MozillaCookieJar()
cj.load('$HOME/.dmacns/cookie.txt', ignore_discard=True)
opener = urllib.request.build_opener(urllib.request.HTTPCookieProcessor(cj))
r = opener.open(urllib.request.Request('http://localhost:18088/api/auth/me'))
print(json.loads(r.read().decode()))
"
```

应返回当前用户信息（did + role + company_name）。

## Common Pitfalls

1. **MSYS bash 下 curl 发中文会乱码** — 所有含中文的请求（搜索、创建商品）必须用 Python `urllib.request`，不能用 `curl`。英文参数可用 curl。
2. **cookie 文件路径** — 统一使用 `~/.dmacns/cookie.txt`。多个 Skill 共享此文件。
3. **角色权限** — 下单需要 `purchase_manager` 或 `operator` 角色。搜商品无限制。如返回权限错误，换对应角色账号登录。
4. **1088 端口冲突** — 如果 18088 被占用，启动时加 `-http :18089` 参数，相应修改 Skill 中的 BASE URL。
5. **$HOME 路径** — 验证脚本中 `$HOME/.dmacns/cookie.txt` 在 MSYS bash 下展开为 `/c/Users/xxx/...`，Python 可正常读写。如果 Agent 用 Windows cmd 而非 bash，需替换为 `C:\\Users\\` 开头的绝对路径。

## Verification Checklist

- [ ] `curl -s http://localhost:18088/api/health` 返回 `"status":"ok"`
- [ ] 登录成功，`~/.dmacns/cookie.txt` 已创建
- [ ] `/api/auth/me` 返回正确的用户信息和角色
- [ ] 知道了当前用户的 did 和 role
