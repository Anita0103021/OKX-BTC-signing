# OKX-BTC-
OKX BTC铭文签名步骤

获取挂单 → 拿到 orderId
    ↓
take 接口 → 拿到 PSBT + 订单ID      (需要 Cookie)
    ↓
本地签名  → 签名写入 tapKeySig       (纯离线，< 5ms)
    ↓
submit   → OKX 验签广播 → txHash    (需要 Cookie)

关键时间窗口： take 到 submit 必须在几秒内完成，否则 OKX 返回 20010018 超时。所以本地签名必须极快（不能等钱包弹窗）。

第一步：获取挂单列表
第二步：获取待签名 PSBT
第三步：本地签名 PSBT（核心）
第四步：提交签名后的 PSBT
