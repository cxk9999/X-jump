# X-jump

将 X(Twitter) 网页链接自动转换为 Twitter Deep Link，并唤醒 iOS 已安装的 X 分身应用。

> 配合 Quantumult X（圈 X）、Loon 或 Egern 使用。
>
> 流程：点击 X 链接 → MitM/Rewrite（307）→ `twitter://` Deep Link → 唤醒 X 分身

## 访问地址

https://cxk9999.github.io/X-jump/

## 支持格式

| 输入链接 | 转换结果 |
| --- | --- |
| `https://x.com/elonmusk` | `twitter://user?screen_name=elonmusk` |
| `https://twitter.com/elonmusk` | `twitter://user?screen_name=elonmusk` |
| `https://x.com/mattshumer_/status/2095609734845927525` | `twitter://status?id=2095609734845927525` |
| `https://twitter.com/mattshumer_/status/2095609734845927525` | `twitter://status?id=2095609734845927525` |

> 说明：仅对「用户主页 / 推文」这两类真实链接做跳转；`/home`、`/messages`、`/i/...` 等功能页与接口**一律不劫持**，保持网页/应用正常打开，避免误伤与跳转循环。

## Quantumult X 配置

### 方式一：远程订阅（推荐）

在圈 X 主配置的 `[rewrite_remote]` 下新增一行：

```ini
[rewrite_remote]
https://cxk9999.github.io/X-jump/qx_rewrite.conf, tag=X-jump, enabled=true
```

远程文件内容即为下方的 rewrite + mitm，后续修改只需更新仓库，圈 X 会自动拉取（更新间隔在圈 X 订阅界面自行设置）。

### 方式二：手动粘贴

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

## Loon 配置

### 方式一：插件（推荐，最省事，已含 MitM）

Loon → 插件 → 右上角 + → 从 URL 添加：

```
https://cxk9999.github.io/X-jump/loon_xjump.plugin
```

插件内已包含 Rewrite 规则 + MitM（`x.com, twitter.com`），安装即用，后续更新在插件列表里刷新即可。

### 方式二：远程重写订阅

Loon 主配置 `[Remote Rewrite]` 下新增一行：

```
https://cxk9999.github.io/X-jump/loon_rewrite.conf, tag=X-jump, enabled=true
```

同时需在 `[MITM]` 中启用：

```
hostname = x.com, twitter.com
```

> ⚠️ 与圈 X 版完全一致的安全原则：只劫持「用户主页 / 推文」，`/i/...`、`/api/...`、功能页、子域名一律放行，不影响 X 应用与网页的正常网络。
>
> 💡 语法使用 Loon 旧版重写语法（`正则 307 twitter://目标`），兼容 Loon 3.x；307 直链不经中间页，且不被 Safari 启发式缓存。

## Egern 配置

Egern 主配置 `modules` 中新增一项（远程模块，更新间隔在 Egern 模块设置界面自行设置）：

```yaml
modules:
  - name: "X-jump"
    url: "https://cxk9999.github.io/X-jump/egern_xjump.module.yaml"
    enabled: true
```

模块内含 `url_rewrites`（4 条 307 深链规则）+ `mitm.hostnames = x.com, twitter.com`。

> ⚠️ 启用前需在 Egern 中安装并信任 MitM 根证书（MitM 仅对 `x.com` / `twitter.com` 两个域名生效，不做子域名泛解析）。
>
> 💡 与圈 X / Loon 版完全一致的安全原则：只劫持主页/推文，`/i/...`、`/api/...`、功能页、子域名一律放行。URL 重写 `status_code: 307` 直接返回重定向到 `twitter://`，不经中间页。

## 部署

- 仓库分支 `main`，站点根目录 `/`（Settings → Pages → Deploy from a branch → main → /(root)）
- 纯静态单文件实现：仅 `index.html`，HTML + JavaScript，无 Node.js / 数据库 / 后端 / 第三方 API

## 注意事项

- 工具能否拦截所有 x.com 点击，取决于 iOS 的 Universal Link 是否在系统层先接管。部署成功后，Rewrite 对 Safari / Telegram 的实际效果仍需真机测试。
- 若 Safari 普通模式不跳转但**无痕模式正常**：是 Safari 缓存了 x.com 的旧响应/旧 302 重定向（登录过网页版 X 后常见）。去 设置 → Safari → 清除历史记录与网站数据（或 高级 → 网站数据 里单独删除 x.com / cxk9999.github.io），即可恢复。
- 若 X 应用存在证书固定（cert pinning），MitM 本身可能使其断网，此时需放弃 MitM 方案。
- `twitter://user?screen_name=...` 与 `twitter://status?id=...` 为已验证可用的 Deep Link 格式。
