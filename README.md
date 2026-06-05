# CalDavZAP

> 源自开源项目 [inf-it.com/open-source/clients/caldavzap/](https://inf-it.com/open-source/clients/caldavzap/)，AGPLv3 协议。

CalDAV 网页客户端，纯静态 JavaScript 单页应用。支持日历事件和任务的查看、创建、编辑、删除。

## 部署

直接上传到任意静态托管服务（Cloudflare Pages、GitHub Pages、Nginx 等）。

```bash
# Cloudflare Pages
wrangler pages deploy . --project-name=caldavzap
```

每次修改文件后，运行 `./cache_update.sh` 更新 HTML5 缓存版本号。

## 使用

1. 打开页面，输入 CalDAV 服务器地址、用户名、密码
2. 点击登录，客户端自动发现服务器上的日历和任务列表
3. 左侧面板选择要显示的日历，主区域查看和管理事件

### 服务器地址格式

不需要带用户名，例如：

- `https://myfreebot.dpdns.org/`
- `https://server.com/caldav.php/`

登录时用户名会自动拼接。

## 配置

编辑 `config.js`，主要配置项：

| 变量 | 说明 |
|------|------|
| `globalTimeZone` | 默认时区 |
| `globalActiveView` | 默认视图：`multiWeek` / `month` / `week` / `day` |
| `globalInterfaceLanguage` | 界面语言 |
| `globalBackgroundSync` | 是否后台同步 |

三种认证模式（互斥）：

- `globalAccountSettings` — 用户名密码写死在 config.js，无登录界面。仅适合内网
- `globalNetworkCheckSettings` — 标准登录界面，用户自行输入凭据（**推荐**）
- `globalNetworkAccountSettings` — 通过 PHP auth 模块代理认证，避免浏览器弹窗

## 技术栈

- jQuery 2.1 + jQuery UI
- FullCalendar（日历视图）
- rrule.js（重复事件展开）
- Spectrum（颜色选择器）

详见 [CLAUDE.md](CLAUDE.md)。
