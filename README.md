# Lunes Host 自动保活脚本

# ⭐ **觉得有用？给个 Star 支持一下！**

> 🧪 注册地址：[https://betadash.lunes.host/](https://betadash.lunes.host/)

定时登录 Lunes Host 并访问服务器页面，保持账号活跃，避免因长时间不活动被暂停。支持 Cloudflare 绕过、Telegram 通知、代理等功能。

---

## ✨ 功能特性

- ✅ 单账号保活  
- ✅ 自动登录（处理 Cloudflare 整页挑战 + Turnstile 验证）  
- ✅ 自动跳转至服务器详情页完成保活  
- ✅ 智能识别“无服务器”场景，仍视为保活成功  
- ✅ 内建速率限制保护（20次/小时阈值自动停止）  
- ✅ 支持 Hysteria2 代理  
- ✅ Telegram 通知（带截图）  
- ✅ 定时 / 手动 / API 多种触发方式  
- ✅ 浏览器状态自动清理，避免 Cookie 干扰  

---

## 📋 前置要求

### 1. GitHub Secrets 配置

进入仓库 `Settings` → `Secrets and variables` → `Actions`，添加以下 Secrets：

| Secret 名称 | 必填 | 说明 | 示例 |
|------------|------|------|------|
| `LUNES` | ✅ | Lunes Host 账号信息 | `邮箱-----密码` |
| `NODE_LINK` | ✅ | 代理链接 | `如 vless:// vmess:// tuic:// hysteria2:// anttls:// socks5://` |
| `TG_BOT_TOKEN` | ❌ | Telegram Bot Token | `123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11` |
| `TG_CHAT_ID` | ❌ | Telegram Chat ID | `123456789` |

### 2. LUNES 格式

**单账号固定格式**：`邮箱-----密码`（五个短横线分隔）

```
your-email@example.com-----your_password_123
```

> ⚠️ 仅支持一个账号。如需多账号管理，请创建多个工作流或仓库。

### 3. 代理格式（确认在v2rayN里使用正常的节点）

`NODE_LINK` 支持以下任意一种代理协议的完整分享链接（不配置则直连）：

- **VLESS**：`vless://uuid@server:port?security=reality&sni=...&type=ws&...`
- **VMess**：`vmess://base64encoded...`
- **Trojan**：`trojan://password@server:port?sni=...&type=ws&...`
- **tuic**：`tuic://uuid:password@server:port...`
- **anytls**：`anytls://uuid@server:port...`
- **hysteria2**：`hysteria2://base64@server:port...`
- **SOCKS5**：`socks5://user:pass@server:port` 或 `socks://user:pass@server:port`

### 4. Telegram 通知配置（可选）

1. 创建 Bot：向 [@BotFather](https://t.me/BotFather) 发送 `/newbot`，获取 Token  
2. 获取 Chat ID：向 [@userinfobot](https://t.me/userinfobot) 发消息或直接与你的 Bot 对话后访问 `https://api.telegram.org/bot<YourToken>/getUpdates`  
3. 将 Bot Token 和 Chat ID 填入 Secrets

---

## 🚀 使用方法

### 方法 1：手动触发（GitHub 网页）

1. 进入仓库 `Actions` 页  
2. 选择 **Lunes Host 自动保活** 工作流  
3. 点击 `Run workflow` → `Run workflow`（绿色按钮）  
4. 运行完成后可在日志和 Telegram 中查看结果

### 方法 2：API 调用

#### 使用 GitHub API 手动触发

```bash
curl -X POST \
  -H "Authorization: Bearer ghp_XXXXXXXXXXXXXXXXXXXXXXXXX" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/你的用户名/你的仓库名/actions/workflows/Lunes-Keep.yml/dispatches \
  -d '{"ref":"main"}'
```

> 替换 `ghp_xxx` 为你自己的 Personal Access Token，需要 `workflow` 权限。

### 方法 3：定时任务

工作流已内置定时执行：

```yaml
schedule:
  # 北京时间每月 1、11、21 日 中午 12:12
  - cron: '12 4 1,11,21 * *'
```

可根据需要修改 cron 表达式，常用示例：

- `0 0 1,15 * *` – 每月 1 日、15 日 0 点（UTC）  
- `12 4 * * *` – 每天 4:12 UTC（即北京时间 12:12）  
- `0 */12 * * *` – 每 12 小时

---

## 📱 Telegram 通知示例

### 保活成功（有服务器）
```
✅ 保活成功
账号：user@example.com
信息：服务器: 12345
时间：2026-06-28 12:14:21

Lunes Host Auto Keep Alive
```
*附带服务器详情页截图*

### 保活成功（无服务器）
```
✅ 保活成功
账号：user@example.com
信息：登录成功，但该账号下没有服务器（可能已被删除）
时间：2026-06-28 12:14:21

Lunes Host Auto Keep Alive
```
*附带控制台截图*

### 保活失败（速率限制）
```
❌ 保活失败
账号：user@example.com
信息：IP 已被限制（Too Many Requests, 20/h），脚本已停止
时间：2026-06-28 12:14:21

Lunes Host Auto Keep Alive
```
*附带速率限制页面截图*

---

## 🔧 高级配置

### 启用截图保存到 Artifacts

默认不上传截图。如需保留截图用于调试，取消 workflow 文件中以下部分的注释：

```yaml
#      - name: 上传截图
#        uses: actions/upload-artifact@v4
#        if: always()
#        with:
#          name: screenshots-${{ github.run_number }}
#          path: output/screenshots/
#          retention-days: 3
```

### 调整保活频率

修改 workflow 中的 `schedule` 字段即可。建议不要低于每 24 小时一次，避免触发 Lunes Host 的频率限制。

### 增加 Cloudflare 绕过重试次数

编辑 `scripts/Lunes-Keep.py`，找到 `bypass_cloudflare_interstitial` 调用，修改 `max_attempts` 参数。

---

## 🐛 常见问题

### 1. 登录失败或卡在 Cloudflare

**原因**：  
- 账号密码错误  
- Turnstile 验证未能自动通过  
- IP 被 Cloudflare 限制  

**解决**：  
- 检查 `LUNES` Secret 格式和账号有效性  
- 在工作流日志中查看截图  
- 尝试配置 Hysteria2 代理更换 IP

### 2. 触发速率限制（Too Many Requests）

**原因**：  
短时间内多次运行脚本，Lunes Host 限制 20 次/小时/ip。  

**解决**：  
- 确保定时任务间隔足够长（推荐 3 天以上）  
- 避免手动频繁触发  
- 使用代理避免与其他人共享 IP

### 3. Telegram 未收到通知

**原因**：  
- Bot Token 或 Chat ID 不正确  
- 未与 Bot 开始对话  

**解决**：  
- 在 Telegram 里向 Bot 发送 `/start`  
- 验证 Secrets 中 `TG_BOT_TOKEN` 和 `TG_CHAT_ID`  
- 查看 Actions 日志中的 `Telegram 通知` 错误提示

---

## 🔒 安全建议

1. ✅ **所有敏感信息存储在 GitHub Secrets** 中，不会打印到日志  
2. ✅ **定期更换账号密码** 并同步更新 Secret  
3. ✅ **限制 Personal Access Token 权限**（仅 `workflow` 或 `repo`）  
4. ✅ **仓库设置为私有** 避免配置文件泄露  
5. ✅ **定期检查 Actions 运行日志**，确保一切正常  

---

## 📄 许可证

MIT License

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

---

**⚠️ 免责声明**：本脚本仅供学习与自动化运维研究，使用者须遵守 Lunes Host 的服务条款。因使用本脚本导致的账号限制或其他问题，作者不承担任何责任。
