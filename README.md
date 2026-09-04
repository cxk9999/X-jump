# X-jump

将 X(Twitter) 网页链接自动转换为 Twitter Deep Link，并唤醒 iOS 已安装的 X 分身应用。

> 配合 Quantumult X（圈 X）、Loon 或 Egern 使用。
>
> 流程：点击 X 链接 → MitM/Rewrite（307）→ `twitter://` Deep Link → 唤醒 X 分身

## 访问地址

[打开 X-jump 跳转页](https://cxk9999.github.io/X-jump/)

X-jump 跳转页（GitHub Pages）：打开后自动把 X 链接解析为 `twitter://` 深链并唤起 X 分身应用，可直接访问测试。

## 支持格式

| 输入链接 | 转换结果 |
| --- | --- |
| `https://x.com/elonmusk` | `twitter://user?screen_name=elonmusk` |
| `https://twitter.com/elonmusk` | `twitter://user?screen_name=elonmusk` |
| `https://x.com/mattshumer_/status/2095609734845927525` | `twitter://status?id=2095609734845927525` |
| `https://twitter.com/mattshumer_/status/2095609734845927525` | `twitter://status?id=2095609734845927525` |

> 说明：仅对「用户主页 / 推文」这两类真实链接做跳转；`/home`、`/messages`、`/i/...` 等功能页与接口**一律不劫持**，保持网页/应用正常打开，避免误伤与跳转循环。

## 订阅地址

### Quantumult X（圈 X）

```
https://cxk9999.github.io/X-jump/qx_rewrite.conf
```

在圈 X 主配置的 `[rewrite_remote]` 下添加该链接即可。

### Loon

```
https://cxk9999.github.io/X-jump/loon_xjump.plugin
```

在 Loon 插件中添加该链接即可（已含 MitM）。也可在 `[Remote Rewrite]` 添加 `https://cxk9999.github.io/X-jump/loon_rewrite.conf` 并在 `[MITM]` 启用 `hostname = x.com, twitter.com`。

### Egern

```
https://cxk9999.github.io/X-jump/egern_xjump.module.yaml
```

在 Egern 模块设置中添加该链接即可（已含 MitM，启用前先安装并信任 MitM 根证书）。

> 以上三个工具功能一致：只劫持「用户主页 / 推文」，307 直链到 `twitter://` 深链；`/i/...`、`/api/...`、功能页、子域名一律放行，不影响 X 正常网络。
> 更新间隔在各工具的订阅/模块设置界面自行设置。

## 手动粘贴（圈 X）

```ini
[rewrite_local]
# 推文：x.com/用户名/status/推文ID → twitter://status?id=ID
^https?:\/\/x\.com\/(?!(?:i|api|home|messages|explore|notifications|search|settings|compose|intent|intents|share|bookmarks|login|signup|tos|privacy|about|jobs|help|download|account|hashtag|lists|people|trends|events|assets|static|oauth|auth)\b)[A-Za-z0-9_]+\/status\/(\d+) url 307 twitter://status?id=$1
^https?:\/\/twitter\.com\/(?!(?:i|api|home|messages|explore|notifications|search|settings|compose|intent|intents|share|bookmarks|login|signup|tos|privacy|about|jobs|help|download|account|hashtag|lists|people|trends|events|assets|static|oauth|auth)\b)[A-Za-z0-9_]+\/status\/(\d+) url 307 twitter://status?id=$1
# 用户主页：x.com/用户名 → twitter://user?screen_name=用户名
^https?:\/\/x\.com\/(?!(?:i|api|home|messages|explore|notifications|search|settings|compose|intent|intents|share|bookmarks|login|signup|tos|privacy|about|jobs|help|download|account|hashtag|lists|people|trends|events|assets|static|oauth|auth)\b)([A-Za-z0-9_]{1,15})\/?(\?.*)?$ url 307 twitter://user?screen_name=$1
^https?:\/\/twitter\.com\/(?!(?:i|api|home|messages|explore|notifications|search|settings|compose|intent|intents|share|bookmarks|login|signup|tos|privacy|about|jobs|help|download|account|hashtag|lists|people|trends|events|assets|static|oauth|auth)\b)([A-Za-z0-9_]{1,15})\/?(\?.*)?$ url 307 twitter://user?screen_name=$1

[mitm]
hostname = x.com,twitter.com
```

> ⚠️ 重要：规则**只能窄不能宽**。不要把 `\/.*` 之类的宽匹配加回来——那会让 X 应用自身的接口请求也被跳转，导致 X 断网。也不建议加 `home/messages` 等功能页规则（若 X 分身是"网页壳"，可能造成跳转循环）。远程订阅版与手动版内容一致。
>
> 💡 采用 `url 307` **直接 307 到 `twitter://` 深链**，不再经过中间页/JS：①307 不被 Safari 启发式缓存；②HTTP 重定向到自定义协议 Safari 允许（JS 非手势跳转会被 Safari 拦截）。

## 部署

- 仓库分支 `main`，站点根目录 `/`（Settings → Pages → Deploy from a branch → main → /(root)）
- 纯静态单文件实现：仅 `index.html`，HTML + JavaScript，无 Node.js / 数据库 / 后端 / 第三方 API

## 注意事项

- 工具能否拦截所有 x.com 点击，取决于 iOS 的 Universal Link 是否在系统层先接管。部署成功后，Rewrite 对 Safari / Telegram 的实际效果仍需真机测试。
- 若 Safari 普通模式不跳转但**无痕模式正常**：是 Safari 缓存了 x.com 的旧响应/旧 302 重定向（登录过网页版 X 后常见）。去 设置 → Safari → 清除历史记录与网站数据（或 高级 → 网站数据 里单独删除 x.com / cxk9999.github.io），即可恢复。
- 若 X 应用存在证书固定（cert pinning），MitM 本身可能使其断网，此时需放弃 MitM 方案。
- `twitter://user?screen_name=...` 与 `twitter://status?id=...` 为已验证可用的 Deep Link 格式。
