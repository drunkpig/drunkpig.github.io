# 通达信量化平台 数据检测报告

> 生成时间：2026-04-21（重新下载数据后更新）
> 测试股票：000001.SZ（平安银行）
> TDX 路径：C:/new_tdx64


## 1. 可用 API 接口（共 64 个）

共 **64** 个公开方法：

**行情获取**：`get_bkjy_value` `get_bkjy_value_by_date` `get_gp_one_data` `get_gpjy_value` `get_gpjy_value_by_date` `get_market_data` `get_market_snapshot` `get_scjy_value` `get_scjy_value_by_date`  
**K线/缓存**：`download_file` `refresh_cache` `refresh_kline`  
**财务/基本面**：`get_divid_factors` `get_financial_data` `get_financial_data_by_date` `get_gb_info` `get_ipo_info` `get_stock_info`  
**板块/自选**：`clear_sector` `create_sector` `delete_sector` `get_sector_list` `get_stock_list_in_sector` `get_user_sector` `rename_sector` `send_user_block`  
**公式选股**：`formula_exp` `formula_format_data` `formula_get_data` `formula_process_mul` `formula_process_mul_xg` `formula_process_mul_zb` `formula_set_data` `formula_set_data_info` `formula_xg` `formula_zb` `tdx_formula`  
**订阅行情**：`get_subscribe_hq_stock_list` `subscribe_hq` `subscribe_quote` `unsubscribe_hq`  
**交易**：`order_stock`  
**其他**：`close` `data_callback_func` `data_transfer` `dll_path` `file_name` `filter_dict_by_fields` `get_cb_info` `get_more_info` `get_stock_list` `get_trackzs_etf_info` `get_trading_calendar` `get_trading_dates` `initialize` `m_is_init_data_transfer` `price_df` `print_to_tdx` `run_id` `run_mode` `send_bt_data` `send_file` `send_message` `send_warn`  


## 2. 各周期 K 线数据深度

> 重新下载数据后更新（2026-04-21），分钟线数据已全部就绪。

| 周期 | 说明 | 最早日期 | 最新日期 | 条数 | 历史深度 |
|------|------|---------|---------|-----:|--------|
| `1m` | 1分钟 | 2026-01-12 | 2026-04-20 | **15,360** | ~3个月 |
| `5m` | 5分钟 | 2026-01-12 | 2026-04-20 | **3,072** | ~3个月 |
| `10m` | 10分钟 | 2026-01-12 | 2026-04-20 | **1,536** | ~3个月 |
| `15m` | 15分钟 | 2026-01-12 | 2026-04-20 | **1,024** | ~3个月 |
| `30m` | 30分钟 | 2026-01-12 | 2026-04-20 | **512** | ~3个月 |
| `1h` | 60分钟 | 2026-01-12 | 2026-04-20 | **256** | ~3个月 |
| `1d` | 日线 | 2021-08-02 | 2026-04-21 | **1,142** | ~5年 |
| `1w` | 周线 | 2021-08-06 | 2026-04-21 | **241** | ~5年 |
| `1mon` | 月线 | 2021-08-31 | 2026-04-21 | **57** | ~5年 |
| `1q` | 季线 | 2021-09-30 | 2026-04-21 | **20** | ~5年 |
| `1y` | 年线 | 2021-12-31 | 2026-04-21 | **6** | ~5年 |
| `45d` | 45日 | 2021-10-12 | 2026-04-21 | **26** | ~5年 |

> **注**：所有分钟周期起点相同（2026-01-12），均由同一份原始1分钟数据聚合而来。
> 日线5年为**服务端硬限制**，重新下载后起点不变，非本地存储问题。


## 3. Tick（分笔）数据支持情况

**测试 `get_market_data(period="tick")`：**

- 结果：❌ 不支持
- 错误码：`-5`
- 错误信息：`周期格式错误：tick（支持['5m', '15m', '30m', '1h', '1d', '1w', '1mon', '1m', '10m', '45d', '1q', '1y']）`

> 实际支持的周期：周期格式错误：tick（支持['5m', '15m', '30m', '1h', '1d', '1w', '1mon', '1m', '10m', '45d', '1q', '1y']）

**其他可能的 Tick 接口：**

相关方法：`get_subscribe_hq_stock_list` `subscribe_hq` `subscribe_quote` `unsubscribe_hq`

`subscribe_hq` / `subscribe_quote` 说明：

- `subscribe_hq`：**实时行情订阅**，非历史数据，仅适合盘中实时使用
- 无历史 tick 批量获取接口，通达信量化平台**不提供历史逐笔数据**


## 4. 数据结构详解

### 4.1 日线 get_market_data(period="1d")

返回字段数：**7**，DataFrame 结构：`index=日期(Timestamp), columns=[股票代码]`

| 字段名 | dtype | 最新值 | 说明 |
|--------|-------|-------|------|
| `Low` | float64 | `10.96` | 最低价 |
| `Amount` | float64 | `69509.72` | 成交额(万元) |
| `High` | float64 | `11.07` | 最高价 |
| `Open` | float64 | `11.01` | 开盘价 |
| `Close` | float64 | `11.06` | 收盘价 |
| `ForwardFactor` | float64 | `1.0` | 前复权因子 |
| `Volume` | float64 | `63003368.0` | 成交量(手) |

**最近5日 Close 数据样本（000001.SZ）：**

```
2026-04-14  11.17
2026-04-15  11.21
2026-04-16  11.09
2026-04-17  11.01
2026-04-20  11.06
```

### 4.2 1分钟线结构

重新下载数据后已可用，结构与日线相同：`index=Timestamp, columns=[股票代码]`，字段同日线（Open/High/Low/Close/Volume/Amount/ForwardFactor）。

- 数据范围：2026-01-12 09:31 ~ 2026-04-20 15:00
- 条数：**15,360** 条（000001.SZ）


## 5. 多股票多市场数据覆盖（日线）

| 股票代码 | 名称 | 市场 | 最早日期 | 最新日期 | 日线条数 |
|---------|------|------|---------|---------|---------|
| 000001.SZ | 平安银行 | 深圳主板 | 2021-08-02 | 2026-04-20 | **1,141** |
| 600519.SH | 贵州茅台 | 上海主板 | 2021-08-02 | 2026-04-20 | **1,140** |
| 300750.SZ | 宁德时代 | 创业板 | 2021-08-02 | 2026-04-20 | **1,141** |
| 688981.SH | 中芯国际 | 科创板 | 2021-08-02 | 2026-04-20 | **1,134** |
| 835974.BJ | 凯华材料 | 北交所 | — | — | ❌ |


## 6. 实时快照 get_market_snapshot

返回字段数：**25**（盘后部分字段为0属正常）

| 字段 | 值 | 说明 |
|------|----|------|
| `ItemNum` | `4055` | 快照笔数 |
| `LastClose` | `11.01` | 昨收 |
| `Open` | `11.01` | 开盘价 |
| `Max` | `11.07` | 最高价 |
| `Min` | `10.96` | 最低价 |
| `Now` | `11.06` | 现价 |
| `Volume` | `630033` | 总手 |
| `NowVol` | `9454` |  |
| `Amount` | `69509.72` | 总成交额(万) |
| `Inside` | `328043` | 内盘 |
| `Outside` | `301991` | 外盘 |
| `TickDiff` | `0.00` | 笔涨跌 |
| `InOutFlag` | `2` |  |
| `Jjjz` | `0.00` |  |
| `Buyp` | `['11.05', '0.00', '0.00', '0.00', '0.00'` | 买一~五价 |
| `Buyv` | `['1001', '0', '0', '0', '0']` | 买量 |
| `Sellp` | `['11.06', '0.00', '0.00', '0.00', '0.00'` | 卖一~五价 |
| `Sellv` | `['9317', '0', '0', '0', '0']` | 卖量 |
| `UpHome` | `0` |  |
| `DownHome` | `0` |  |
| `Before5MinNow` | `11.05` | 5分钟前价 |
| `Average` | `11.03` | 均价 |
| `XsFlag` | `2` |  |
| `Zangsu` | `0.09` | 涨速 |
| `ZAFPre3` | `-1.34` | 3日涨幅 |


## 7. 股票列表 get_stock_list

- `market="5"` (全A股)：**5516 只**，样例：['000001.SZ', '000002.SZ', '000004.SZ']
- `market=""` (沪深全市场)：**5516 只**，样例：['000001.SZ', '000002.SZ', '000004.SZ']
- `market="0"` (深圳)：**3 只**，样例：['999999.SH', '001896.SZ', '601127.SH']
- `market="1"` (上海)：**0 只**，样例：[]
- `market="2"` (北京)：**0 只**，样例：[]


## 8. 财务数据 get_financial_data

- 调用时服务端返回空数据（`None`），推测需要特定版本或行情权限
- 正确签名：`get_financial_data(stock_list, field_list, start_time, end_time, report_type='report_time')`（无 `table_list` 参数）


## 9. 与 miniQMT（东莞证券版）对比

### 9.1 数据深度对比

| 维度 | 通达信 TdxQuant | miniQMT（东莞证券） |
|------|----------------|-------------------|
| **日线历史** | 2021-08-02 起（~5年，服务端限制） | 2020-01 起（~6年，更长） |
| **分钟线历史** | 2026-01-12 起（~3个月） | 2026-04-01 起（~3周） |
| **Tick 历史** | ❌ 完全不支持历史获取 | ✅ 2026-03-20 起（~1个月） |
| **Tick 字段** | 无 | lastPrice/open/high/low/volume/amount/askPrice/bidPrice 等20字段 |

### 9.2 功能覆盖对比

| 功能 | 通达信 TdxQuant | miniQMT（东莞证券） |
|------|----------------|-------------------|
| **市场覆盖** | 沪深京 + 港股 + 美股 + 期货 + ETF + 宏观 | 沪深京（A股为主） |
| **API 数量** | 64个 | 较少，主要围绕行情和交易 |
| **实时快照** | ✅ `get_market_snapshot`（26字段，含五档盘口） | ✅ `get_market_data` 实时档位 |
| **历史 Tick** | ❌ 不支持 | ✅ 支持（约1个月） |
| **分钟线下载** | ✅ 客户端盘后下载 | ✅ `download_history_data` 拉取 |
| **公式选股** | ✅ 调用通达信公式（`formula_*` 系列） | ❌ 无 |
| **财务数据** | ⚠️ 接口存在但返回空（权限问题） | ❌ 无 |
| **板块/自选股** | ✅ 完整板块操作 | ❌ 无 |
| **交易下单** | ✅ `order_stock` | ✅ 支持实盘交易 |
| **需要客户端** | ✅ 必须后台运行通达信终端 | ✅ 必须后台运行 miniQMT |
| **费用** | 免费（终端免费版） | 免费（需开户东莞证券） |

### 9.3 选型建议

| 场景 | 推荐 | 理由 |
|------|------|------|
| 日线回测 | **miniQMT** | 历史更长（6年 vs 5年） |
| 分钟线回测 | **TdxQuant** | 历史更长（3个月 vs 3周） |
| Tick 回测 | **miniQMT** | TdxQuant 完全不支持 |
| 实时监控选股 | **TdxQuant** | 公式选股 + 板块操作更完整 |
| 多市场（港美期货） | **TdxQuant** | miniQMT 主要覆盖A股 |
| 实盘交易 | 均可 | 视券商账户情况 |


---


## 10. 示例代码

### 10.0 TDX MCP — 获取大盘1分钟K线（直连行情服务器，无需终端）

TDX MCP（`~/.claude/mcp-servers/tdx_mcp`）直连通达信行情服务器，**无需通达信终端在后台运行**，可实时获取大盘1分钟K线（含今日盘中数据）。

```python
# ── 通过 MCP 工具调用（Claude Code 内）─────────────
# market=1(上海), code=999999(上证综指), period=7(1分钟)
# times=3 → 约2400根（约10个交易日），不传count/start（有bug）
mcp__tdx__stock_kline(market=1, code="999999", period=7, times=3)

# 其他常用大盘指数：
# 399001 market=0  深证成指
# 399006 market=0  创业板指
# 000300 market=1  沪深300
```

**直接调底层 API（Python 脚本中使用）：**

```python
import sys
sys.path.insert(0, '/c/Users/cxu/.claude/mcp-servers/tdx_mcp')
from opentdx.client.quotationClient import QuotationClient
from opentdx.const import MARKET, PERIOD, ADJUST

client = QuotationClient(auto_retry=True)
client.login()

# 大盘1分钟K，最近2400根（约10交易日），含今日实时
bars = client.get_kline(
    market=MARKET.SH,
    code='999999',
    period=PERIOD.MIN_1,   # =7
    count=2400,
    adjust=ADJUST.NONE
)
# bars: list[dict]，每条含 datetime/open/high/low/close/vol/amount/turnover
# 大盘额外含 up_count/down_count（涨跌家数）

# 转 DataFrame
import pandas as pd
df = pd.DataFrame(bars)
df['datetime'] = pd.to_datetime(df['datetime'])
df.set_index('datetime', inplace=True)
print(df.tail(5))
```

**返回数据结构：**

```
datetime               open      close     high      low       vol          up_count  down_count
2026-04-21 13:58:00    4081.35   4081.30   4081.64   4080.71   22184946     829       1456
2026-04-21 13:59:00    4081.22   4079.78   4081.23   4079.22   26172890     805       1485
2026-04-21 14:00:00    4079.81   4078.92   4080.04   4078.92   26238444     792       1500
```

**与 TdxQuant 的关键差异：**

| 项目 | TDX MCP (opentdx) | TdxQuant |
|------|-------------------|----------|
| 需要终端运行 | ❌ 不需要 | ✅ 必须 |
| 大盘指数1分K | ✅ | ✅ |
| 历史深度（1分K） | ~10交易日（times=3） | ~3个月 |
| 今日实时数据 | ✅ 当前分钟 | ✅ |

> **结论**：需要长历史分钟回测用 TdxQuant；仅需近期或实时大盘监控用 TDX MCP 更轻量。

---

### 10.1 通达信 TdxQuant — 获取 Bar 数据

```python
import sys
sys.path.insert(0, 'C:/new_tdx64/PYPlugins/user')
from tqcenter import tq

tq.initialize(__file__)  # 需要通达信终端在后台运行

# ── 日线（000001.SZ 最近60天）──────────────────────
d = tq.get_market_data(
    field_list=[],            # 空=返回全部字段
    stock_list=['000001.SZ'],
    period='1d',
    start_time='20260101',
    end_time='20260421',
    count=-1,                 # -1=返回区间全部数据
    dividend_type='front',    # 前复权
    fill_data=False
)
# 返回结构：dict，key=字段名，value=DataFrame(index=Timestamp日期, columns=[股票代码])
close_df = d['Close']           # shape=(N, 1)
print(close_df.tail(5))

# ── 1分钟线（多股票批量）──────────────────────────
d1m = tq.get_market_data(
    field_list=['Open', 'High', 'Low', 'Close', 'Volume'],
    stock_list=['000001.SZ', '600519.SH'],
    period='1m',
    start_time='20260420',
    end_time='20260421',
    count=-1,
    dividend_type='none',
    fill_data=False
)
# 取平安银行1分钟收盘价
close_1m = d1m['Close']['000001.SZ']
print(close_1m.tail(10))

# ── 实时快照（盘中使用）───────────────────────────
snap = tq.get_market_snapshot(stock_code='000001.SZ', field_list=[])
print(f"现价={snap['Now']}  最高={snap['Max']}  最低={snap['Min']}")
print(f"买一: {snap['Buyp'][0]} × {snap['Buyv'][0]}手")
print(f"卖一: {snap['Sellp'][0]} × {snap['Sellv'][0]}手")

# ── 实时 Tick 订阅（仅盘中，无历史）──────────────
def on_tick(datas):
    for code, data in datas.items():
        print(f"[TICK] {code} 现价={data.get('lastPrice')} 成交量={data.get('volume')}")

tq.subscribe_hq(stock_list=['000001.SZ'], callback=on_tick)
# 注意：subscribe_hq 是异步回调，需在事件循环中保持运行
```

### 10.2 miniQMT（东莞证券）— 获取 Bar 数据

```python
import sys
sys.path.insert(0, 'D:/workspace2/bs-point')
from xtquant import xtdata

# miniQMT 无需显式登录，自动连接已运行的 QMT 客户端

# ── 日线（下载 + 读取）────────────────────────────
code = '000001.SZ'
xtdata.download_history_data(code, period='1d',
                             start_time='20200101', end_time='20260421')

d = xtdata.get_market_data(
    field_list=['time', 'open', 'high', 'low', 'close', 'volume'],
    stock_list=[code],
    period='1d',
    start_time='20200101',
    end_time='20260421',
    count=-1,
    dividend_type='none',
    fill_data=True
)
# 返回结构：dict，key=字段名，value=DataFrame(index=[stock_code], columns=时间戳)
close = d['close'].loc[code]   # Series，index=时间戳(int)
print(f"条数={len(close)}  最早={close.index[0]}  最新={close.index[-1]}")

# ── 1分钟线（下载 + 读取）────────────────────────
xtdata.download_history_data(code, period='1m',
                             start_time='20260401', end_time='20260421')

d1m = xtdata.get_market_data(
    field_list=['open', 'high', 'low', 'close', 'volume'],
    stock_list=[code],
    period='1m',
    start_time='20260401',
    end_time='20260421',
    count=-1,
    dividend_type='none',
    fill_data=False
)
close_1m = d1m['close'].loc[code]
print(close_1m.tail(5))

# ── Tick 历史（下载 + 读取）──────────────────────
xtdata.download_history_data(code, period='tick',
                             start_time='20260320', end_time='20260421')

d_tick = xtdata.get_local_data(
    field_list=[],
    stock_list=[code],
    period='tick',
    start_time='20260320',
    end_time='20260421',
    count=-1
)
df_tick = d_tick[code]   # DataFrame，index=时间字符串(YYYYMMDDHHmmss)
# 字段：time/lastPrice/open/high/low/lastClose/amount/volume/
#       askPrice(list)/bidPrice(list)/askVol(list)/bidVol(list) 等20字段
print(f"Tick条数={len(df_tick)}")
print(df_tick[['lastPrice','volume','askPrice','bidPrice']].tail(3))

# ── 实时 Tick 订阅──────────────────────────────
def on_tick(datas):
    for code, ticks in datas.items():
        for tick in ticks:
            print(f"[TICK] {code} 价={tick['lastPrice']} 量={tick['volume']}")

xtdata.subscribe_quote(code, period='tick', count=0, callback=on_tick)
xtdata.run()  # 阻塞，保持订阅
```

### 10.3 两者数据结构差异对照

| 对比项 | TdxQuant | miniQMT |
|--------|---------|---------|
| **返回结构** | `dict{字段: DataFrame(index=日期, col=股票)}` | `dict{字段: DataFrame(index=股票, col=时间戳)}` |
| **日期轴** | `index` 为 `Timestamp` | `columns` 为 `int` 时间戳 |
| **取单股数据** | `d['Close']['000001.SZ']`（按列） | `d['close'].loc['000001.SZ']`（按行） |
| **下载方式** | 客户端盘后下载，无需代码触发 | `xtdata.download_history_data()` 代码触发 |
| **Tick 历史** | 不支持 | `get_local_data(period='tick')` |
| **复权参数** | `dividend_type='front'/'back'/'none'` | `dividend_type='front'/'back'/'none'` |


---


## 11. 总结与回测建议

### 数据深度汇总

| 周期 | 历史深度 | 可用性 |
|------|---------|--------|
| 日线(1d) | 2021-08-02 至 2026-04-21，共 1,142 条 | ✅ |
| 周线(1w) | 2021-08-06 至 2026-04-21，共 241 条 | ✅ |
| 月线(1mon) | 2021-08-31 至 2026-04-21，共 57 条 | ✅ |
| 1分钟(1m) | 2026-01-12 至 2026-04-20，共 15,360 条 | ✅ |
| 5分钟(5m) | 2026-01-12 至 2026-04-20，共 3,072 条 | ✅ |
| 15分钟(15m) | 2026-01-12 至 2026-04-20，共 1,024 条 | ✅ |
| 30分钟(30m) | 2026-01-12 至 2026-04-20，共 512 条 | ✅ |
| 60分钟(1h) | 2026-01-12 至 2026-04-20，共 256 条 | ✅ |
| Tick(逐笔) | ❌ 不支持历史获取 | 仅实时订阅 |

### 关键结论

1. **Tick 历史**：`get_market_data` 不支持 `period="tick"`，通达信量化平台**无历史逐笔数据接口**
   - 实时 tick 用 `subscribe_hq` / `subscribe_quote` 订阅，仅适合盘中实盘使用
   - **回测无法用 tick 粒度**

2. **日线**：历史从 **2021-08-02** 起，共 **1,142** 条，是最完整的回测数据源（5年为服务端硬限制）
3. **分钟线**：重新下载后全部可用，历史从 **2026-01-12** 起（约3个月），`1m~1h` 均由同一份原始数据聚合

### 回测策略建议

| 策略类型 | 建议周期 | 备注 |
|---------|---------|------|
| 趋势/选股 | `1d` | 数据最完整，历史最长 |
| 短线T+0 | `1m`/`5m` | ✅ 已可用，历史约3个月（2026-01起） |
| 高频/微观 | Tick | ❌ 历史数据不可用，仅实盘 |