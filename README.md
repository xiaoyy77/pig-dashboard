# 生猪出栏跟踪面板

牧原 / 温氏 / 德康 三家企业日出栏数据的监控面板。
线上地址：https://xiaoyy77.github.io/pig-dashboard/

## 文件

| 文件 | 作用 | 是否手改 |
|---|---|---|
| `index.html` | 页面：布局、走势图、环比与累计计算 | 不动 |
| `data.js` | **唯一数据文件**，由解析脚本生成 | 由脚本覆写 |
| `_tools/xlsx_to_data.py` | Excel → `data.js` 解析脚本（本地，不发布） | 需要时改 |
| `_tools/verify.py` | 无头 Chrome 渲染验证（本地，不发布） | 需要时改 |

## 数据口径（重要）

三家的原始字段并不统一，页面按各自口径展示：

- **牧原**：`MY外销合计`（头）。另有 17 个分省列，但 2025 年期间**分省列之和 ≠ 合计列**
  （435 天里 158 天不等，例如 2025-04-01 合计 64,434 vs 分省之和 83,771），
  2026-09 起两者相等。页面以**合计列**为准，并在分省表里同时显示两个数。
- **温氏**：日度只有**江苏、安徽**两地的出栏量，总计 = 两者相加。
  （仅 2025-04-30 一天原表总计与两地之和不符）
- **德康**：日度历史上是**订购数量**（头）+ 均重 + 计划 + 计划完成率；
  当日分区域明细里另有**计划出栏、实际订购、订购率、出栏价、预估均重**。

## 更新流程

1. 把新的 Excel 发给 Hermes Agent
2. 解析：
   ```bash
   cd ~/pig-dashboard
   uv run --with openpyxl python _tools/xlsx_to_data.py \
     "C:/Users/YingX/Downloads/mywsdk.xlsx" \
     "C:/Users/YingX/Downloads/9月14号dk出栏.xlsx" \
     "2026-09-14" "C:/Users/YingX/pig-dashboard/data.js"
   ```
   参数：主表、德康当日明细、明细日期、输出文件
3. 验证渲染：`python _tools/verify.py`
4. 生成预览图给用户确认
5. `git add -A && git commit && git push` → 线上约 30 秒刷新，网址不变

## 数据格式（data.js）

```js
window.PIG_DATA = {
  meta: {updated, source, isDemo, units},
  companies: [{id, name, color, asOf, sub}],   // asOf = 该公司最新数据日
  provinces: [...],                            // 牧原省份列顺序
  daily: [{
    date: "2026-09-04",
    muyuan: {total, prov:{河南:5750, 山东:10791, ...}},
    wens:   {js, ah, total},
    dekang: {ordered, weight, plan, fulfill, fromDetail?}
  }],
  dkDetail: {
    date, total: {plan, ordered, rateCalc, price, weight, nBatch, nRegion},
    regions: [{name, orderRate, plan, ordered, rateCalc, price, weight,
               nBatch, nFarm, quotes,
               batches: [{farm, breed, plan, price, weight, quotes}]}]
  }
};
```

环比、当周累计、当月累计**全部由页面实时计算**，数据里不预存。

## 本地预览

直接双击 `index.html`（数据用 `<script src>` 加载，所以 `file://` 下也能用）。
