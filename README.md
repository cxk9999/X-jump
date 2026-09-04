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
^https?:\/\/x\.com\/.* url 302 https://cxk9999.github.io/X-jump/?url=$0
^https?:\/\/twitter\.com\/.* url 302 https://cxk9999.github.io/X-jump/?url=$0

[mitm]
hostname = x.com,*.x.com,twitter.com,*.twitter.com
```

## 部署

- 仓库分支 `main`，站点根目录 `/`（Settings → Pages → Deploy from a branch → main → /(root)）
- 纯静态单文件实现：仅 `index.html`，HTML + JavaScript，无 Node.js / 数据库 / 后端 / 第三方 API

## 注意事项

- 圈 X 能否拦截所有 x.com 点击，取决于 iOS 的 Universal Link 是否在系统层先接管。Pages 部署成功后，Rewrite 对 Safari / Telegram 的实际效果仍需真机测试。
- `twitter://user?screen_name=...` 与 `twitter://status?id=...` 为已验证可用的 Deep Link 格式。
