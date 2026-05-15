# OKX-BTC 铭文签名流程

OKX BTC 铭文挂单的本地签名与提交方案，核心目标：**在几秒内完成 take → 签名 → submit 全流程**，避免 OKX 返回超时错误（错误码 `20010018`）。

---

## 流程概览

```
获取挂单列表
     ↓
take 接口  →  拿到 PSBT + 订单ID        (需要 Cookie)
     ↓
本地签名   →  签名写入 tapKeySig         (纯离线，< 5ms)
     ↓
submit    →  OKX 验签广播  →  txHash    (需要 Cookie)
```

> ⚠️ **关键时间窗口**：`take` 到 `submit` 必须在几秒内完成，否则 OKX 返回 `20010018` 超时。
> 本地签名必须极快，**不能等钱包弹窗**。

---

## 步骤详解

### 第一步：获取挂单列表

调用 OKX 挂单列表接口，获取目标 `orderId`。

```js
// 示例：拉取 BTC 铭文挂单
const orders = await getOrderList({ type: 'inscription', coin: 'BTC' })
const orderId = orders[0].orderId
```

---

### 第二步：获取待签名 PSBT

使用 `take` 接口，传入 `orderId`，返回待签名的 PSBT 和订单信息。

```js
// 需要携带有效 Cookie
const { psbt, orderId } = await take({ orderId })
```

**请求需要：**
- 有效的登录 Cookie
- 正确的 `orderId`

---

### 第三步：本地签名 PSBT（核心）

对 PSBT 进行本地离线签名，将签名结果写入 `tapKeySig` 字段。

```js
// 纯离线操作，< 5ms
const signedPsbt = localSign(psbt, privateKey)
// 签名写入 tapKeySig
```

**要求：**
- 必须使用本地私钥直接签名
- 禁止调用外部钱包（会引入弹窗延迟）
- 签名耗时需控制在 **5ms 以内**

---

### 第四步：提交签名后的 PSBT

将签名完成的 PSBT 通过 `submit` 接口提交给 OKX，完成验签和链上广播。

```js
// 需要携带有效 Cookie
const { txHash } = await submit({ orderId, signedPsbt })
console.log('广播成功:', txHash)
```

**返回：**
- `txHash`：链上交易哈希

---

## 错误码参考

| 错误码 | 含义 | 解决方案 |
|--------|------|----------|
| `20010018` | take → submit 超时 | 优化签名速度，确保全流程在时间窗口内完成 |

---

## 注意事项

- Cookie 需保持有效，`take` 和 `submit` 均依赖登录态
- 本地签名逻辑需妥善保管私钥，避免泄露
- 建议在正式运行前用测试网验证流程完整性
