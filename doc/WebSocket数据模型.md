# Website WebSocket 数据模型

## 文件位置

| 文件 | 用途 |
|------|------|
| `src/utils/ws.js` | WebSocket 基类（连接管理、心跳、订阅） |
| `src/store/sfutures.js` | 合约交易 WebSocket 逻辑 |
| `src/store/stocks.js` | 股票交易 WebSocket 逻辑 |
| `src/api/stocks.js` | 股票行情 WebSocket（Ondo/Market） |

## WebSocket 连接地址

| 连接 | URL | 用途 |
|------|-----|------|
| 行情 WS | `wss://{host}/openapi/quote/ws/v1` | 合约行情 |
| 用户 WS | `wss://{host}/api/ws/user` | 合约/股票用户数据 |
| Amber WS | `wss://amber.stockcoin.ai/ws/connect` | 股票行情（Ondo） |
| Market WS | `wss://{host}/ws` | 股票行情 |

---

## 通用消息信封结构

所有 WebSocket 消息遵循以下结构：

| 字段 | 类型 | 说明 |
|------|------|------|
| `topic` | string | 主题标识符 |
| `channel` | string | 备选主题标识符（可选） |
| `name` | string | 订阅名称（部分消息） |
| `event` | string | 操作类型：`"sub"` / `"unsub"` / `"cancel"` |
| `id` | string | 请求追踪 ID |
| `code` | number | 响应码（200 = 成功） |
| `type` | string | 消息类型：`"snapshot"` / `"diff"` / `"SUBSCRIBE_SUCCESS"` / `"PONG"` |
| `data` | any | 消息负载（数组、对象或基本类型） |
| `f` | boolean | 快照标志（K线数据） |
| `params` | object | 附加参数 |
| `symbol` | string | 交易对标识符 |
| `ts` | number | 时间戳 |
| `op` | string | 操作类型 |

---

## 一、行情 WebSocket 数据模型（Quote WS）

### 1.1 mergedDepth — 深度/盘口

**订阅请求：**

| 字段 | 值 |
|------|-----|
| `event` | `"sub"` |
| `topic` | `"mergedDepth"` |
| `symbol` | `"301.{coin}"` |
| `id` | `"depth301.{coin}.{dumpScale}"` |
| `limit` | `22` |
| `params.dumpScale` | 精度档位 |
| `params.binary` | 是否压缩 |

**返回数据 `data.data[0]` 字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `s` | string | 交易对名称 |
| `a` | array | 卖盘，每项为 `[price, size]` |
| `b` | array | 买盘，每项为 `[price, size]` |
| `spread` | number | 买卖价差 |

**前端处理后结构：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `bids[].px` | number | 买单价格 |
| `bids[].size` | number | 买单数量 |
| `bids[].total` | number | 累计数量 |
| `asks[].px` | number | 卖单价格 |
| `asks[].size` | number | 卖单数量 |
| `asks[].total` | number | 累计数量 |
| `spread` | number | 价差 |
| `symbol` | string | 交易对 |
| `timestamp` | number | 时间戳 |

---

### 1.2 trade — 最近成交

**订阅请求：**

| 字段 | 值 |
|------|-----|
| `event` | `"sub"` |
| `topic` | `"trade"` |
| `symbol` | `"301.{coin}"` |
| `limit` | `50` |
| `params.org` | `6066` |

**返回数据 `data.data[]` 每项字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `t` | number | 成交时间戳 |
| `p` | number | 成交价格 |
| `q` | number | 成交数量 |
| `m` | boolean | 方向标志，`true`=买入，`false`=卖出 |
| `s` | string | 交易对 |

**前端处理后结构：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `time` | number | 时间戳 |
| `px` | number | 价格 |
| `sz` | number | 成交额（USD） |
| `side` | string | `"buy"` / `"sell"` |
| `age` | string | 格式化时间 |

---

### 1.3 index — 指数价格

**订阅请求：**

| 字段 | 值 |
|------|-----|
| `event` | `"sub"` |
| `topic` | `"index"` |
| `symbol` | 从 coin 中提取的报价币种（如 `"USDT"`） |

**返回数据 `data.data[]` 每项字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `symbol` | string | 指数名称 |
| `index` | number | 指数价格 |

---

### 1.4 markPrice — 标记价格

**订阅请求：**

| 字段 | 值 |
|------|-----|
| `event` | `"sub"` |
| `topic` | `"markPrice"` |
| `symbol` | `"301.{coin}"` |
| `id` | `"markPrice_301.{coin}"` |

**返回数据 `data.data` 字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `symbolId` | string | 交易对 ID |
| `price` | number | 标记价格 |

---

### 1.5 realtimes / slowBroker — 24h 行情统计

**订阅请求：**

| 字段 | 值 |
|------|-----|
| `event` | `"sub"` |
| `topic` | `"realtimes"` |
| `symbol` | `"301.{coin}"` |
| `params.org` | `6066` |
| `params.realtimeInterval` | `"24h"` |

**返回数据 `data.data[]` 每项字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `s` | string | 交易对名称（如 `"AAPL-PERP-USDT"`） |
| `c` | number | 收盘价/最新价 |
| `v` | number | 成交量 |
| `qv` | number | 成交额 |
| `m` | number | 涨跌幅 |
| `o` | number | 开盘价 |

**前端处理后结构：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `name` | string | 交易对名称 |
| `dayNtlVlm` | number | 24h 成交额 |
| `dayQty` | number | 24h 成交量 |
| `latestPrice` | string | 最新价（保留2位小数） |
| `margin` | number | 涨跌幅 |
| `openPrice` | number | 开盘价 |
| `isDown` | boolean | 是否下跌 |

---

### 1.6 kline_{interval} — K 线数据

**订阅请求：**

| 字段 | 值 |
|------|-----|
| `event` | `"sub"` |
| `topic` | `"kline_{interval}"`（如 `"kline_1h"`） |
| `symbol` | `"301.{coin}"` |
| `params.klineType` | 周期（`"1h"` / `"1d"` / `"1w"` 等） |
| `params.limit` | `500` 或 `1500` |

**返回数据 `data.data` 每项字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `t` | number | K线时间戳 |
| `ts` | number | 备选时间戳 |
| `o` | number | 开盘价 |
| `h` | number | 最高价 |
| `l` | number | 最低价 |
| `c` | number | 收盘价 |
| `v` | number | 成交量 |
| `i` | string | 周期标识 |

**说明：** `data.f === true` 时为快照数据（全量），否则为增量更新。

**前端处理后结构：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `time` | number | 时间戳 |
| `open` | number | 开盘价 |
| `high` | number | 最高价 |
| `low` | number | 最低价 |
| `close` | number | 收盘价 |
| `volume` | number | 成交量 |

---

## 二、用户 WebSocket 数据模型（User WS）

### 2.1 futures_position — 合约持仓

**订阅请求：**

| 字段 | 值 |
|------|-----|
| `event` | `"sub"` |
| `topic` | `"futures_position"` |
| `id` | `"futures_position"` |

**返回数据 `data.data[]` 每项字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `symbolId` | string | 交易对 ID |
| `symbolName` | string | 交易对名称 |
| `avgPrice` | number | 平均开仓价 |
| `isLong` | string | 方向，`"1"`=多头，`"0"`=空头 |
| `positionValues` | number | 仓位价值（USD） |
| `total` | number | 持仓数量 |
| `margin` | number | 已用保证金 |
| `markPrice` | number | 当前标记价格 |
| `liquidationPrice` | number | 强平价格 |

**前端计算字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `size` | number | 持仓规模 = 数量 × 合约乘数 |
| `unrealisedPnl` | number | 未实现盈亏 = (标记价 - 开仓价) × 规模 × 方向 |
| `profitRate` | number | 收益率 = 未实现盈亏 / 保证金 |
| `positionValue` | number | 仓位价值 = 规模 × 标记价 |
| `maintainMargin` | number | 维持保证金 |
| `riskRate` | number | 风险率 |

---

### 2.2 futures_balance — 合约资产余额

**订阅请求：**

| 字段 | 值 |
|------|-----|
| `event` | `"sub"` |
| `topic` | `"futures_balance"` |
| `id` | `"futures_balance"` |

**返回数据 `data.data[]` 每项字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `tokenId` | string | 币种 ID（如 `"USDT"`） |
| `availableMargin` | number | 可用保证金 |
| `crossUnRealizedPnl` | number | 全仓未实现盈亏 |

**前端计算字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `availableOpenMargin` | number | 可用开仓保证金 = max(0, availableMargin + min(0, crossUnRealizedPnl)) |

---

### 2.3 futures_order — 合约委托

**订阅请求：**

| 字段 | 值 |
|------|-----|
| `event` | `"sub"` |
| `topic` | `"futures_order"` |
| `id` | `"futures_order"` |

**返回数据 `data.data[]` 每项字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `orderId` | string | 订单 ID |
| `symbolId` | string | 交易对 ID |
| `status` | string | 状态：`"NEW"` / `"PARTIALLY_FILLED"` / `"FILLED"` / `"CANCELED"` |
| `time` | number | 下单时间戳 |
| `side` | string | 买卖方向 |
| `price` | number | 委托价格 |
| `quantity` | number | 委托数量 |
| `filledQuantity` | number | 已成交数量 |
| `slOrderId` | string | 关联止损单 ID |
| `spOrderId` | string | 关联止盈单 ID |

**说明：** `data.type === "snapshot"` 为全量快照，`"diff"` 为增量更新。

---

### 2.4 futures_plan_order — 条件单/止盈止损

**订阅请求：**

| 字段 | 值 |
|------|-----|
| `event` | `"sub"` |
| `topic` | `"futures_plan_order"` |
| `id` | `"sub_futures_plan_order_all"` |

**返回数据 `data.data[]` 每项字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `orderId` | string | 条件单 ID |
| `status` | string | 状态：`"UNTREATED"` / `"TRIGGERED"` / `"CANCELLED"` / `"EXPIRED"` / `"REJECTED"` |
| `time` | number | 创建时间戳 |
| 其他字段 | — | 与普通委托类似 |

---

### 2.5 futures_order_filled — 合约成交

**订阅请求：**

| 字段 | 值 |
|------|-----|
| `event` | `"sub"` |
| `topic` | `"futures_order_filled"` |
| `id` | `"futures_order_filled"` |

**返回数据：** 结构与 `futures_order` 相同，表示已成交的订单。

---

### 2.6 futures_tradeable — 可交易资产

**订阅请求：**

| 字段 | 值 |
|------|-----|
| `event` | `"sub"` |
| `topic` | `"futures_tradeable"` |
| `id` | `"futures_tradeable"` |

**返回数据 `data.data[]`：** 可交易资产列表。

---

## 三、股票 WebSocket 数据模型

### 3.1 balance — 股票资产余额

**订阅请求：**

| 字段 | 值 |
|------|-----|
| `event` | `"sub"` |
| `topic` | `"balance"` |
| `id` | `"balance"` |
| `params.org` | `6066` |

**返回数据 `data.data[]` 每项字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `tokenId` | string | 币种 ID |

---

### 3.2 order — 股票委托

**订阅请求：**

| 字段 | 值 |
|------|-----|
| `event` | `"sub"` |
| `topic` | `"order"` |
| `id` | `"order"` |
| `params.org` | `6066` |

**返回数据 `data.data[]` 每项字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `orderId` | string | 订单 ID |
| `symbolId` | string | 交易对 ID |
| `time` | number | 下单时间戳 |

---

### 3.3 order_filled — 股票成交

**订阅请求：**

| 字段 | 值 |
|------|-----|
| `event` | `"sub"` |
| `topic` | `"order_filled"` |
| `id` | `"order_filled"` |
| `params.org` | `6066` |

**返回数据：** 结构与 `order` 相同。

---

## 四、股票行情 K 线（独立 WS）

### 4.1 Ondo K 线

**连接地址：** `wss://amber.stockcoin.ai/ws/connect`

**订阅请求：**

| 字段 | 值 |
|------|-----|
| `op` | `"subscribe"` |
| `name` | `"market.kline.updated"` |
| `payload.filter` | `["eth:{symbol}:USD:{interval}"]` |

**返回数据 `data.payload.data` 字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `time` | number | K线时间戳 |
| `open` | number | 开盘价 |
| `high` | number | 最高价 |
| `low` | number | 最低价 |
| `close` | number | 收盘价 |
| `vol` | number | 成交量 |

### 4.2 Market K 线

**连接地址：** `wss://{host}/ws`

**订阅请求：**

| 字段 | 值 |
|------|-----|
| `type` | `"SUBSCRIBE"` |
| `data.topic` | `"KLINE:{coin}:{interval}"` |
| `timestamp` | 当前时间戳 |

**返回数据 `data.data` 字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `time` | number | K线时间戳 |
| `open` | number | 开盘价 |
| `high` | number | 最高价 |
| `low` | number | 最低价 |
| `close` | number | 收盘价 |
| `vol` | number | 成交量 |

