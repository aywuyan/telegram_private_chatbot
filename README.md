# 🤖 Telegram Private Chatbot (v5.4) 

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/jikssha/telegram_private_chatbot)
![GitHub stars](https://img.shields.io/github/stars/jikssha/telegram_private_chatbot?style=social)
![License](https://img.shields.io/badge/License-MIT-blue.svg)
[![Telegram](https://img.shields.io/badge/Telegram-DM-blue?style=social&logo=telegram)](https://t.me/vaghr_wegram_bot)
[🇺🇸 English](README_EN.md) | [🇨🇳 简体中文](README.md)

**Telegram Private Chatbot** 是一个基于 **Cloudflare Workers** 的高性能 Telegram 双向私聊机器人。它专为解决 Telegram 上的垃圾广告骚扰而生，拥有 Cloudflare Turnstile 人机验证、智能内容过滤系统、强大的管理员指令集以及无缝的消息转发体验。

无需购买服务器，利用 Cloudflare 强大的边缘计算网络，即可免费部署一套企业级的客户服务系统。

---

<details>
<summary>📢 <b>v5.4 版本重要更新公告 (2026-06-06)</b></summary>
   
### 新增功能：
- **☁️ Cloudflare Turnstile 人机验证**：支持 Turnstile 点击验证，比传统答题更难被绕过，完全免费（每月 100 万次）。配置 `TURNSTILE_SITE_KEY`、`TURNSTILE_SECRET_KEY`、`VERIFICATION_PAGE_URL` 三个环境变量即可启用。
- **🔍 智能垃圾内容过滤**：支持关键词检测（`SPAM_KEYWORDS` 环境变量）、链接拦截（新用户 24 小时内禁止发链接）、重复消息熔断（相同内容 3 次自动拦截）。
- **🛡️ 骚扰消息通知**：检测到疑似骚扰消息时自动通知管理员群组，支持静默丢弃模式（`SPAM_SILENCE_MODE`）。
- **📄 内置验证页面**：Turnstile 验证页面直接内嵌在 Worker 中，无需额外部署 Pages 项目。
- **📋 `/help` 指令**：在话题中输入 `/help` 可查看所有管理员指令列表，无需翻看文档。

### 改进：
- 未配置 Turnstile 时自动降级为本地题库验证，兼容旧版本部署
- 垃圾检测在 `handlePrivateMessage` 和 `forwardToTopic` 双重执行，防止并发绕过
- Turnstile 验证通过后自动删除验证消息，聊天界面更干净
- 验证通过提示优化为简洁设计风格
- `/info` 指令增加可点击跳转用户私聊（手机端）
- `/cleanup` 可在任意话题执行，不再受限

### ⚠️ 更新指南：
Fork 用户可直接点击 sync 更新同步，自动更新
手动部署用户复制 worker.js 代码到 worker，重新部署一次，并添加新的环境变量
</details>

---

<details>
<summary>📢 <b>v5.1 版本重要更新公告 (2026-01-05)</b></summary>
   
### 主要修复：
- **自动话题修复**：被删除话题用户不再转发到 General，会自动新建话题。
- **话题无限循环修复**：针对创建失败添加重试机制，最多重试 3 次。
- **消息路由规范化**：修复字符串与数字混用问题，统一规范化为 String 类型。
- **并发验证加固**：添加验证锁机制，彻底杜绝并发绕过漏洞。
- **数据读取保护**：实现 `safeGetJSON()` 安全读取机制，防止 KV 数据损坏导致崩溃。
- **验证系统重构**：改用索引方案，完全避免按钮回调截断问题，100% 可用。
   
### 更新功能：
**批量清理工具**：/cleanup  # 扫描并清理已删除话题的用户数据

### ⚠️ 更新指南：
Fork用户可直接点击sync 更新同步，自动更新
手动部署用户复制worker.js代码到worker,重新部署一次
</details>

---

## 📑 目录 (Table of Contents)

* [✨ 核心特性](#-核心特性)
* [🛠️ 管理员指令](#-管理员指令)
* [🚀 部署教程](#-部署教程)
    * [方法一：GitHub 一键连接 (推荐)](#方法一github-一键连接部署-推荐-)
    * [方法二：手动复制部署](#方法二手动复制部署-简单直接)
    * [最后一步：激活 Webhook](#最后一步激活-webhook-至关重要)
* [❓ 常见问题 (FAQ)](#-常见问题-faq)
* [📈 Star History](#-star-history)

---

## ✨ 核心特性

v4.0 版本移除了所有不稳定的外部 API 依赖，专注于**极致的速度**与**绝对的稳定性**。

| 特性 | 描述 |
| :--- | :--- |
| **⚡ 0 延迟验证** | 采用**本地精选常识题库**。秒开秒验，彻底告别网络超时与接口报错，验证成功率 100%。**v5.4 新增可选 Turnstile 点选验证**，防机器人能力更强。 |
| **🛡️ 智能防骚扰** | **短 ID 机制**修复了 Telegram 按钮点击失效的 Bug。验证通过后提供 **30 天免打扰期**，兼顾安全与用户体验。**v5.4 新增关键词过滤、链接拦截、重复消息检测**。 |
| **💬 话题群组管理** | 利用 **Telegram Forum Topics** 功能，自动为每位私聊用户创建一个独立的话题，消息隔离，管理井井有条。 |
| **👮 隐形指令系统** | 自动**拦截**用户端发送的 `/` 开头指令，防止普通用户骚扰管理员。管理指令仅在管理员群组内生效。 |
| **🔒 权限控制** | 强大的指令集：支持 **封禁 (/ban)**、**解封 (/unban)**、**结单 (/close)** 和 **永久信任 (/trust)** 等操作。 |
| **☁️ Serverless** | 完全基于 Cloudflare Workers 运行。**0 成本**、无需服务器、无需运维、抗高并发。 |
| **📸 多媒体支持** | 完美支持文本、图片、视频、文件等多种消息格式的双向转发，不丢失任何细节。 |

---

## 🛠️ 管理员指令

> **注意**：以下指令仅在 **管理员群组的话题内** 有效。用户在私聊窗口发送指令会被静默拦截，不会对管理员造成骚扰。

| 指令 | 作用 | 适用场景 |
| :--- | :--- | :--- |
| `/close` | **强制关闭对话**<br>机器人会提示用户对话已结束，并拒收新消息。 | 工单处理完成，礼貌结束咨询。 |
| `/open` | **重新开启对话**<br>恢复对该用户的消息转发。 | 误操作关闭，或用户需再次联系。 |
| `/ban` | **封禁用户**<br>机器人将完全无视该用户的所有消息（无提示）。 | 遇到恶意刷屏、广告机器人。 |
| `/unban` | **解封用户**<br>恢复该用户的正常通讯权限。 | 给予改过自新的机会。 |
| `/trust` | **永久信任**<br>该用户将永久免除人机验证（永不过期）。 | 熟人、VIP 客户、长期合作伙伴。 |
| `/reset` | **重置验证**<br>强制清除该用户的验证状态，下次需重新验证。 | 测试验证流程，或怀疑账号被盗。 |
| `/info` | **查看信息**<br>显示当前用户的 UID、话题 ID 和验证状态。 | 查询用户资料。 |
| `/help` | **帮助**<br>显示所有管理员指令列表。 | 忘记指令时快速查阅。 |
| `/cleanup` | **批量清理**<br>扫描并清理已删除话题的用户数据。 | 清理失效用户。 |

---

## 🛡️ 反骚扰系统说明 (v5.4 新增)

### Cloudflare Turnstile 人机验证

> **推荐启用**：Turnstile 是 Cloudflare 提供的免费 CAPTCHA 替代方案，用户只需点击一个复选框即可验证，不能被自动化脚本绕过。

**启用方式**：
1. 前往 [Cloudflare Dashboard](https://dash.cloudflare.com/) → **Turnstile** → **Add Site**
2. 创建一个 Turnstile Widget，获取 `Site Key`（公开）和 `Secret Key`（私有）
3. 在 Worker 的 **Settings** → **Variables** 中添加以下环境变量：

| 变量名 | 值 |
|--------|-----|
| `TURNSTILE_SITE_KEY` | 你的 Turnstile Site Key |
| `TURNSTILE_SECRET_KEY` | 你的 Turnstile Secret Key |
| `VERIFICATION_PAGE_URL` | Worker 的完整 URL（如 `https://xxx.workers.dev`） |

> 💡 **免费额度**：Turnstile 每月提供 **100 万次**验证，对于个人使用完全免费。

> 💡 **平滑降级**：如果不配置以上变量，机器人会自动使用本地题库进行人机验证。

### 垃圾内容过滤

通过 `SPAM_KEYWORDS` 环境变量配置广告关键词，多个关键词用**逗号、分号或换行**分隔。

**示例**：
```
SPAM_KEYWORDS = 加微信,引流,免费领,点击链接,加我QQ,日赚,兼职,代理,招代理,免费送,限时优惠,扫码加,私聊我,付费咨询
```

系统会对**已通过验证的用户**的消息进行三重检测：
1. **关键词匹配** — 消息中包含 `SPAM_KEYWORDS` 中的任意关键词，直接拦截
2. **链接拦截** — 新验证用户（24 小时内）发送链接时自动拦截
3. **重复消息熔断** — 同一用户短时间内重复发送相同内容超过 3 次，自动拦截

**拦截行为**：
- 被拦截的消息**不会转发到群组**（管理员不会被打扰）
- 在管理员群组**发送通知**（含用户 ID、拦截原因），方便管理员判断是否需要 `/ban`
- 可选启用 `SPAM_SILENCE_MODE` 环境变量（设为 `true`）来完全静默处理

---

## 🚀 部署教程

### 前置准备
1.  **Telegram Bot**：找 [@BotFather](https://t.me/BotFather) 申请一个机器人，获取 `Token`。
    * *重要设置*：在 BotFather 中关闭 **Group Privacy** (`/mybots` > Settings > Group Privacy > Turn off)。
2.  **管理员群组**：创建一个 Telegram 群组，并**开启话题功能 (Topics)**。
    * 将机器人拉入群组，并设为**管理员**（给予管理话题权限）。
    * 获取群组 ID（通常以 `-100` 开头）。
     ``获取 SUPERGROUP_ID 小技巧：
在 Telegram 桌面端右键群内任意消息，复制消息链接；链接里会有一段 -100xxxxxxxxxx 或 xxxxxxxxxx；若只看到纯数字 xxxxxxxxxx，在前面加上 -100，就是完整的 SUPERGROUP_ID（私密频道/群组同理）。``

### 方法一：GitHub 一键连接部署 (推荐 ★)

这是最简单的自动化部署方式，当您更新 GitHub 仓库时，Cloudflare 会自动重新部署您的 Worker。

1.  **Fork 本仓库** 到您的 GitHub 账户。
2.  登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)。
3.  导航到 **Workers & Pages** -> **Create Application**。
4.  点击 **Connect to Git** 标签页。
5.  授权 Cloudflare 访问您的 GitHub，并选择您刚才 Fork 的 `telegram_private_chatbot` 仓库。
6.  **配置部署设置**：
    * 项目名称：`telegram-private-chatbot` (或任意名称)。
    * 生产分支：通常是 `main` 或 `master`。
    * 其余保持默认，点击 **Save and Deploy**。
7.  **⚠️ 关键步骤：绑定数据库与变量**
    * 部署完成后，进入该 Worker 的 **Settings** -> **Variables** 页面。
    * **绑定 KV 数据库** (必须)：
        * 在 Cloudflare 左侧菜单 **KV** 中创建一个新的 Namespace（例如叫 `TOPIC_MAP`）。
        * 回到 Worker 的 Variables 页面，向下滚动到 **KV Namespace Bindings**。
        * 点击 **Add binding**，变量名填写 `TOPIC_MAP` (必须全大写)，Namespace 选择刚才创建的那个。
    * **添加环境变量**：
        * `BOT_TOKEN`: 你的机器人 Token。
        * `SUPERGROUP_ID`: 你的群组 ID (例如 -100123...)。
    * **（推荐）添加反骚扰变量**（详见 [反骚扰系统说明](#-反骚扰系统说明-v54-新增)）：
        * `TURNSTILE_SITE_KEY`、`TURNSTILE_SECRET_KEY`、`VERIFICATION_PAGE_URL`（启用 Turnstile）
        * `SPAM_KEYWORDS`：广告关键词（启用内容过滤）
8.  **最后一步**：配置完成后，点击页面顶部的 **Deployments** 标签，找到最新的部署记录，点击右侧的 **Retry deployment** (重新部署)，让变量生效。

### 方法二：手动复制部署 (简单直接)

如果您不想关联 GitHub，可以直接复制代码。

1.  登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)。
2.  进入 **Workers & Pages** -> **Create Application** -> **Create Worker** ，选择从`hello world`开始。
3.  命名你的 Worker，点击 **Deploy**。
4.  点击 **Edit code**，将本项目 `worker.js` 的所有代码复制粘贴进去，覆盖原代码。
5.  点击右上角 **Deploy** 保存。
6.  **配置 KV 与变量**：
    * 去 **Settings** -> **Variables**。
    * 添加 KV 绑定：Variable name 填 `TOPIC_MAP`，并绑定一个 KV 数据库。
    * 添加**环境变量**：
        * `BOT_TOKEN`: 你的机器人 Token。
        * `SUPERGROUP_ID`: 你的群组 ID (例如 -100123...)。
    * **（推荐）添加反骚扰环境变量**（详见 [反骚扰系统说明](#-反骚扰系统说明-v54-新增)）：
        * `TURNSTILE_SITE_KEY`: Cloudflare Turnstile Site Key（可选）
        * `TURNSTILE_SECRET_KEY`: Cloudflare Turnstile Secret Key（可选）
        * `VERIFICATION_PAGE_URL`: Worker 的完整 URL（可选）
        * `SPAM_KEYWORDS`: 广告关键词，逗号分隔（可选）
    * 点击 **Save and Deploy**。

---

### 最后一步：激活 Webhook (至关重要)

无论使用哪种部署方式，最后都需要手动告诉 Telegram 你的 Worker 地址。请在浏览器中**严格按顺序**访问以下 URL：

 **设置新 Webhook**：
    ```
   (https://api.telegram.org/bot)<YOUR_TOKEN>/setWebhook?url=<YOUR_WORKER_URL>
    ```
    *将 `<YOUR_TOKEN>` 替换为机器人 Token，`<YOUR_WORKER_URL>` 替换为 Worker 的完整域名或者你绑定的自定义的域名 (如 `https://xxx.workers.dev`)。*
    
 *举例：https://api.telegram.org/bot1234:HUSH2GW/setWebhook?url=https://1234.workers.dev* `<YOUR_TOKEN>前面的bot别删了`

如果返回 `{"ok":true, "result":true, "description":"Webhook was set"}`，即表示部署成功！

---

## ❓ 常见问题 (FAQ)

**Q1: 为什么点击验证按钮没有反应？**
A: 请检查 Webhook 是否正确设置。必须确保 Telegram 允许发送 `callback_query` 事件。请务必执行上述“最后一步”中的重置操作。

**Q2: 为什么机器人无法在群里创建话题？**
A: 请确保：1. 群组 ID 正确（-100开头）；2. 群组已开启 Topics 功能；3. 机器人是群管理员且拥有 "Manage Topics" 权限。

**Q3: 如何启用 Cloudflare Turnstile 人机验证？**
A: 前往 Cloudflare Dashboard → Turnstile 创建站点 → 获取 Site Key 和 Secret Key → 在 Worker 环境变量中设置 `TURNSTILE_SITE_KEY`、`TURNSTILE_SECRET_KEY`、`VERIFICATION_PAGE_URL`（Worker 自身的 URL）。具体参考 [反骚扰系统说明](#-反骚扰系统说明-v54-新增)。

**Q4: Turnstile 和本地题库可以同时使用吗？**
A: 如果配置了 Turnstile 相关变量，机器人会优先使用 Turnstile；如果没配置，会自动降级为本地题库验证。两者不需要同时开启。

**Q6: 为什么人机验证能通过收不到转发的消息？**
A: 请仔细检查所有变量名称和id是否准确，删除webhook再重新激活。
 `(https://api.telegram.org/bot)<YOUR_TOKEN>/deleteWebhook?drop_pending_updates=true` 
  
  如果依然无法正常转发消息，尝试完成所有步骤后，最后再添加bot的管理员权限。

**Q7: 为什么会检测到骚扰消息？**
A: 系统通过三重机制过滤骚扰消息：1) 关键词匹配（`SPAM_KEYWORDS` 环境变量配置的广告关键词）；2) 新用户链接拦截（24小时内禁止发链接）；3) 重复消息熔断（相同内容重复3次）。被拦截的消息不会转发到群组，但会在群组发送通知。管理员可以使用 `/trust` 将正常用户设为永久信任，免除此类检测。
  
**Q5: 为什么webhook设置失败？**
A: 如果你设置了自定义域名不成功，Webhook 改回 workers.dev 域名再尝试。这种情况是你域名解析失败或者网络环境阻断造成的
 
---

## 🔒 安全说明

> [!IMPORTANT]
> 请妥善保管您的 Bot API Token 和 Turnstile Secret Key，不要泄露。`TURNSTILE_SITE_KEY` 是唯一可以公开的反骚扰变量（仅用于前端页面渲染），`TURNSTILE_SECRET_KEY` 必须保密。

---

## 📈 Star History

[![Star History Chart](https://api.star-history.com/svg?repos=jikssha/telegram_private_chatbot&type=date&legend=top-left)](https://www.star-history.com/#jikssha/telegram_private_chatbot&type=date&legend=top-left)

---
**如果这个项目对你有帮助，请给个 Star ⭐️ 吧！**
