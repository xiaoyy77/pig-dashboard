# 生猪日度数据面板

牧原 / 温氏 / 德康 三家企业日度数据监控面板。

## 文件说明

| 文件 | 作用 | 谁改 |
|---|---|---|
| `index.html` | 页面：布局、走势图、环比与累计计算逻辑 | 不用动 |
| `data.js` | **唯一的数据文件** | 每次更新时覆写 |

页面与数据分离，所以更新数据不会动到页面代码，网址永久不变。

## 数据格式

`data.js` 内容为 `window.PIG_DATA = {...}`，结构：

```js
{
  "meta": {
    "updated": "2026-09-14 10:30",     // 页面右上角显示
    "source": "涌益 / 公司公告 / …",     // 页脚显示
    "isDemo": false,                    // true 时页面顶部显示“示例数据”提示
    "units": {"price":"元/公斤", "weight":"公斤/头", "volume":"头"}
  },
  "companies": [
    {"id":"muyuan", "name":"牧原", "color":"#C0392B"},
    {"id":"wens",   "name":"温氏", "color":"#1F6F8B"},
    {"id":"dekang", "name":"德康", "color":"#B7791F"}
  ],
  "daily": [
    {"date":"2026-09-12",
     "muyuan": {"price":12.85, "weight":121.0, "volume":173000},
     "wens":   {"price":13.41, "weight":124.0, "volume": 79000},
     "dekang": {"price":13.66, "weight":127.3, "volume": 26000}}
  ]
}
```

- `daily` 按日期升序；缺失值写 `null`（页面显示 `—`，不会算崩）。
- `volume` 单位是**头**，页面自动换算成万头。
- 环比、月累计、加权均价全部由页面实时计算，数据里不用预存。

## 更新流程

1. 把数据发给 Hermes Agent（Excel / CSV / 文本 / 截图均可）
2. Agent 解析后覆写 `data.js`，生成预览图给你确认
3. 确认后 `git push`，线上页面自动刷新（约 30 秒），网址不变

## 本地预览

直接双击 `index.html` 即可，无需服务器。
