# X-jump

将 X(Twitter) 网页链接自动转换为 Twitter Deep Link，并唤醒 iOS 已安装的 X 分身应用。

> 配合 Quantumult X（圈 X）MitM + Rewrite 使用。
>
> 流程：点击 X 链接 → Quantumult X Rewrite → 跳转 GitHub Pages → 解析原始链接 → 转换为 `twitter://` Deep Link → 唤醒 X 分身

## 访问地址

https://cxk9999.github.io/X-jump/

## 支持格式

| 输入链接 | 转换结果 |
| --- | --- |
| `https://x.com/elonmusk` | `twitter://user?screen_name=elonmusk` |
| `https://twitter.com/elonmusk` | `twitter://user?screen_name=elonmusk` |
| `https://x.com/mattshumer_/status/2095609734845927525` | `twitter://status?id=2095609734845927525` |
| `https://twitter.com/mattshumer_/status/2095609734845927525` | `twitter://status?id=2095609734845927525` |
| 其他未知链接（`/home`、`/messages`、`/i/bookmarks` 等） | `twitter://`（直接打开 X 首页） |

## Quantumult X 配置

### 方式一：远程订阅（推荐）

在圈 X 主配置的 `[rewrite_remote]` 下新增一行：

```ini
[rewrite_remote]
https://cxk9999.github.io/X-jump/qx_rewrite.conf, tag=X-jump, update-interval=86400, enabled=true
```

远程文件内容即为下方的 rewrite + mitm，后续修改只需更新仓库，圈 X 会自动拉取。

### 方式二：手动粘贴

```ini
[rewrite_local]
# 推文：x.com/用户名/status/推文ID（仅数字ID，用户名段排除保留路径）
^https?:\/\/x\.com\/(?!(?:i|api|home|messages|explore|notifications|search|settings|compose|intent|intents|share|bookmarks|login|signup|tos|privacy|about|jobs|help|download|account|hashtag|lists|people|trends|events|assets|static|oauth|auth)\b)[A-Za-z0-9_]+\/status\/\d+ url 307 https://cxk9999.github.io/X-jump/?url=$0
^https?:\/\/twitter\.com\/(?!(?:i|api|home|messages|explore|notifications|search|settings|compose|intent|intents|share|bookmarks|login|signup|tos|privacy|about|jobs|help|download|account|hashtag|lists|people|trends|events|assets|static|oauth|auth)\b)[A-Za-z0-9_]+\/status\/\d+ url 307 https://cxk9999.github.io/X-jump/?url=$0
# 用户主页：x.com/用户名（单段路径，排除 i/api/home/messages 等保留路径）
^https?:\/\/x\.com\/(?!(?:i|api|home|messages|explore|notifications|search|settings|compose|intent|intents|share|bookmarks|login|signup|tos|privacy|about|jobs|help|download|account|hashtag|lists|people|trends|events|assets|static|oauth|auth)\b)[A-Za-z0-9_]{1,15}\/?(\?.*)?$ url 307 https://cxk9999.github.io/X-jump/?url=$0
^https?:\/\/twitter\.com\/(?!(?:i|api|home|messages|explore|notifications|search|settings|compose|intent|intents|share|bookmarks|login|signup|tos|privacy|about|jobs|help|download|account|hashtag|lists|people|trends|events|assets|static|oauth|auth)\b)[A-Za-z0-9_]{1,15}\/?(\?.*)?$ url 307 https://cxk9999.github.io/X-jump/?url=$0

[mitm]
# 只对 x.com / twitter.com 两个域名本身启用 MitM，不做 *. 泛解析，避免误伤接口子域名
hostname = x.com,twitter.com
```

> ⚠️ 重要：规则**只能窄不能宽**。不要把 `\/.*` 之类的宽匹配加回来——那会让 X 应用自身的接口请求也被跳转，导致 X 断网。远程订阅版与手动版内容一致。
>
> 💡 使用 `url 307` 而非 `302`：Safari 会启发式缓存 302 重定向，导致再次点击同一链接时命中旧缓存（表现为"Safari 不行、Chrome 正常"）。307 每次强制重新请求，可规避该问题。

## 部署

- 仓库分支 `main`，站点根目录 `/`（Settings → Pages → Deploy from a branch → main → /(root)）
- 纯静态单文件实现：仅 `index.html`，HTML + JavaScript，无 Node.js / 数据库 / 后端 / 第三方 API

## 注意事项

- 圈 X 能否拦截所有 x.com 点击，取决于 iOS 的 Universal Link 是否在系统层先接管。Pages 部署成功后，Rewrite 对 Safari / Telegram 的实际效果仍需真机测试。
- 若 X 应用存在证书固定（cert pinning），MitM 本身可能使其断网，此时需放弃 MitM 方案。
- `twitter://user?screen_name=...` 与 `twitter://status?id=...` 为已验证可用的 Deep Link 格式。
