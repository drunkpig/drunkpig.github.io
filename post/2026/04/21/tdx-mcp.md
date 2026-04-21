# 通达信 MCP 调研报告

> 生成时间：2026-04-21
> MCP 路径：`~/.claude/mcp-servers/tdx_mcp`
> 底层库：自研 `opentdx`（非 pytdx），直连通达信行情服务器（TCP 7709/7727端口）

---

## 1. 可用工具列表

| 工具名 | 说明 |
|--------|------|
| `get_index_overview` | 大盘6指数实时快照（无参数） |
| `stock_kline` | 股票/指数 K 线（含1分钟线） |
| `stock_tick_chart` | 盘中分时数据（今日或历史某天） |
| `stock_top_board` | 板块涨跌榜 |
| `stock_quotes_list` | 分类行情列表（含排序过滤） |
| `stock_quotes` | 个股/多股实时行情 |
| `stock_unusual` | 异动股票 |
| `stock_auction` | 集合竞价数据 |
| `stock_transaction` | 分时成交（今日逐笔） |
| `stock_f10` | F10基本面数据 |
| `goods_*` 系列 | 期货/商品行情（同股票系列） |
| `indicator_*` 系列 | 技术指标计算（MA/EMA/MACD/RSI/KDJ/BOLL/ATR/VolMA） |

---

## 2. K 线周期对照表

来源：`opentdx/const.py` PERIOD 枚举

| period 值 | 说明 | 备注 |
|-----------|------|------|
| **7** | **1分钟** | ✅ 实测可用，含今日实时 |
| 0 | 5分钟 | ✅ 实测可用 |
| 1 | 15分钟 | — |
| 2 | 30分钟 | — |
| 3 | 60分钟 | — |
| **4** | **日线** | ✅ 实测可用 |
| 5 | 周线 | ✅ 实测可用 |
| 6 | 月线 | — |
| 8 | 多分钟（配合 `times` 参数，如10分钟=`times=10`） | 底层支持 |
| 9 | 多日（配合 `times`，如45日） | 底层支持 |
| 10 | 季线 | — |
| 11 | 年线 | — |
| 13 | 多秒（如5秒/15秒，配合 `times`） | 底层支持，MCP未暴露 |

**大盘1分钟K确认可用**：`market=1, code=999999, period=7` 返回含今日实时数据。

---

## 3. stock_kline 参数说明与已知 Bug

### 函数签名（底层 get_kline）

```python
get_kline(market, code, period, start=0, count=800, times=1, adjust=ADJUST.NONE)
```

### MCP 参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `market` | int | 必填 | 0=深圳, 1=上海, 2=北京 |
| `code` | str | 必填 | 股票/指数代码 |
| `period` | int | 必填 | 周期，见上表 |
| `times` | int | 1 | 数据量倍数（times=3 → 2400根） |
| `count` | str | "800" | ⚠️ 有bug，不能传，只能用默认 |
| `start` | str | "0" | ⚠️ 有bug，不能传，只能用默认 |
| `adjust_type` | str | "none" | 复权：none/qfq/hfq |

### ⚠️ count/start Bug

MCP schema 将 `count`/`start` 定义为 **string 类型**，但底层做了 `int < str` 的比较，导致只要传入这两个参数就报错：

```
'<' not supported between instances of 'int' and 'str'
```

**解决方案**：不传 `count`/`start`，改用 `times` 倍数控制数据量：
- 默认 `times=1` → 800根（约3.3个交易日的1分钟数据）
- `times=3` → 2400根（约10个交易日）

### K 线数据结构

```json
{
  "datetime": "2026-04-21T14:00:00",
  "open": 4079.807,
  "close": 4078.922,
  "high": 4080.037,
  "low": 4078.922,
  "vol": 26238444.0,      // 成交量（手）
  "amount": 2623845376.0, // 成交额（元）
  "turnover": 5.46,       // 换手率（%）
  // 指数额外字段：
  "up_count": 792,        // 上涨家数
  "down_count": 1500      // 下跌家数
}
```

---

## 4. get_index_overview 数据结构

无参数，返回6个主要指数：

| code | 指数 | market |
|------|------|--------|
| 999999 | 上证综指 | 1 |
| 399001 | 深证成指 | 0 |
| 399006 | 创业板指 | 0 |
| 899050 | 北证A50 | 2 |
| 000688 | 科创50 | 1 |
| 000300 | 沪深300 | 1 |

返回字段：`market/code/active/pre_close/diff/close/open/high/low/vol/amount/up_count/down_count`

---

## 5. stock_transaction 分时成交结构

今日逐笔聚合数据（每个价格档位一条），支持传 `date` 查历史：

```json
{
  "time": "09:30:00",
  "price": 1412,
  "vol": 95,        // 成交量（手）
  "trans": 29,      // 成交笔数
  "action": "BUY",  // BUY / SELL / NEUTRAL（集合竞价）
  "unknown": 0
}
```

---

## 6. stock_tick_chart 分时快照结构

返回今日盘中每分钟快照（无时间戳字段，按顺序排列）：

```json
{
  "price": 4076.33,   // 当前价
  "avg": 4069.902,    // 均价
  "vol": 3534420      // 该分钟成交量
}
```

注：无 OHLC，仅现价+均价+量，约240条（一天交易分钟数）。

---

## 7. 与 TdxQuant（通达信量化平台）对比

| 维度 | TDX MCP（opentdx） | TdxQuant（tqcenter） |
|------|--------------------|----------------------|
| **接入方式** | 直连行情服务器 TCP | 需通达信终端后台运行 |
| **1分钟K线** | ✅ 实时，约10交易日（times=3） | ✅ 约3个月（2026-01起） |
| **历史深度（日线）** | ~800个交易日 | 服务端限制约5年 |
| **大盘指数** | ✅ 999999/399001等 | ✅ 同 |
| **Tick历史** | ✅ stock_transaction（今日+历史某天） | ❌ 无历史，仅实时订阅 |
| **多秒K线** | 底层支持 period=13 | 不支持 |
| **财务/F10** | ✅ stock_f10 | ⚠️ 接口存在但返回空 |
| **技术指标** | ✅ 内置MA/MACD/KDJ等 | ❌ 无 |
| **板块数据** | ✅ stock_top_board | ✅ 完整板块操作 |

---

## 8. 使用示例

### 获取今日大盘1分钟K（最近2400根）

```python
# 通过 MCP 调用
mcp__tdx__stock_kline(market=1, code="999999", period=7, times=3)
```

### 直接用底层 API

```python
from opentdx.client.quotationClient import QuotationClient
from opentdx.const import MARKET, PERIOD, ADJUST

client = QuotationClient(auto_retry=True)
client.login()

# 大盘1分钟K，最近2400根
bars = client.get_kline(
    market=MARKET.SH,
    code="999999",
    period=PERIOD.MIN_1,
    count=2400,
    adjust=ADJUST.NONE
)

# 个股日线（前复权）
bars = client.get_kline(
    market=MARKET.SZ,
    code="000001",
    period=PERIOD.DAILY,
    count=500,
    adjust=ADJUST.QFQ
)
```
