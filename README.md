# 生猪出栏跟踪面板

牧原 / 温氏 / 德康 三家企业日出栏数据监控面板。
线上地址：https://xiaoyy77.github.io/pig-dashboard/

> **本页面已加密。** `data.js` 里只有 AES-256 密文，没有访问密码打不开页面。
> 密码由维护人持有，向需要查看的人口头/微信告知即可。

## 文件

| 文件 | 作用 | 是否手改 |
|---|---|---|
| `index.html` | 页面：布局、解密、环比与累计计算 | 不动 |
| `data.js` | **加密后的数据**（AES-GCM-256 + PBKDF2-150000） | 由脚本生成 |
| `_tools/xlsx_to_data.py` | Excel → 明文数据（本地，不发布） | 需要时改 |
| `_tools/crypt_data.mjs` | 明文 → 加密 `data.js`；密码管理 | 一般不动 |
| `_tools/pw_check.py` | Playwright 端到端验证解锁与渲染 | 一般不动 |
| `_tools/.secret` | **访问密码**（已 gitignore，绝不上传） | 不要提交 |
| `_tools/data.plain.js` | 明文数据（已 gitignore，便于增量更新） | 不要提交 |

桌面还有一份密码副本：`生猪面板密码.txt`（看完可删）。

## 更新流程

```bash
cd ~/pig-dashboard

# 1. 解析 Excel -> 明文数据
uv run --with openpyxl python _tools/xlsx_to_data.py   "<主表.xlsx>" "<德康当日明细.xlsx>" "<明细日期>" "_tools/data.plain.js" "2026-03-01"

# 2. 明文 -> 加密 data.js
node _tools/crypt_data.mjs

# 3. 验证解密链路
node _tools/crypt_data.mjs --verify

# 4. 浏览器端到端验证（锁屏 / 解锁 / 免密重进 / 错误密码）
uv run --with playwright python _tools/pw_check.py

# 5. 发布
git add -A && git commit -m "更新数据至 <日期>" && git push
```

## 密码相关

- 查看密码：打开桌面 `生猪面板密码.txt`，或 `cat _tools/.secret`
- 换密码：`node _tools/crypt_data.mjs --new-password` → 重新加密 → 推送
  （换完旧密码立即失效；同事需重新输入）
- 忘记密码：删掉 `_tools/.secret` 重新生成，然后重新加密发布（数据不丢，明文在 `_tools/data.plain.js`）

## 安全边界（重要）

- 数据用 **AES-GCM-256** 加密，密钥由密码经 **PBKDF2 15 万次** 派生，密码本身不存进网页。
- 同时访问 `data.js` 也只能拿到密文，不会泄露数据。
- 保护强度**取决于密码强度**：用长随机密码；不要用 `123456` 这类。
- 所有拿到密码的人都能看到数据，无法按人区分权限或单独吊销。
  若需要「每人独立账号、可单独禁用」，得改用 Cloudflare Access 之类带身份验证的方案。
- `_tools/` 整个目录已 gitignore，密码与明文数据不会进仓库。

## 数据口径

- **牧原**：`MY外销合计`（头），2026-03 起分省为河南/山东/河北/山西，合计与分省之和一致。
- **温氏**：江苏 + 安徽两地之和。
- **德康**：日度为**订购数量**（头）；当日分区域明细含计划出栏、实际订购、订购率、出栏价、预估均重。
- **当周累计周期按各家**：牧原 周日→周五；温氏 周六→周五；德康 周一→周日。
- 口径与源表核对过：牧原 09-04 = 157,466 头、温氏 09-04 = 141,000 头，与 Excel 完全一致。

## 本地预览

直接双击 `index.html`，输入密码即可（数据用 `<script src>` 加载，`file://` 下也可用）。
