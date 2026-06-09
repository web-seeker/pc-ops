---
name: token-security-audit
description: Token 安全检测与隐私保护。当用户提到 Token、API Key、密钥、密码、隐私、安全检测、token泄露检测时自动触发。用于检查 Token 是否泄露、安全使用最佳实践、隐私风险评估。
---

# Token 安全检测与隐私保护助手

## 概述

当检测到 Token、API Key、密钥等敏感信息相关问题时，本技能将帮助用户：
- 检测 Token 是否已被记录或泄露
- 评估隐私风险等级
- 提供安全处理建议
- 指导正确的 Token 管理实践

---

## 使用场景

### 触发关键词
- "Token 安全吗？"
- "会泄露吗？"
- "怎么检测泄露？"
- "Token 注意事项"
- "API Key 隐私"
- "密钥安全管理"

### 常见问题
1. 在聊天中发送了 Token，担心泄露
2. 工具要求提供 Token，不知道是否安全
3. 想检查本地是否有 Token 被记录
4. 不知道如何安全管理 Token

---

## 检测流程

### 第一步：检查日志文件

检查以下常见日志位置是否包含 Token：

```powershell
# PowerShell 检查日志
Get-ChildItem -Path "$env:USERPROFILE" -Recurse -Filter "*.log" -ErrorAction SilentlyContinue | 
    Select-String -Pattern "TOKEN_PATTERN" | Select-Object Path
```

### 第二步：检查 GitHub 授权

访问以下页面检查异常授权：
- https://github.com/settings/applications
- https://github.com/settings/tokens

### 第三步：检查浏览器扩展

检查已安装的浏览器扩展，移除可疑或不需要的扩展。

---

## 风险等级评估

| 等级 | 说明 | 颜色 |
|------|------|------|
| 🟢 低风险 | Token 未被成功使用，仅出现在错误日志中 | 绿色 |
| 🟡 中等风险 | Token 被记录在多个位置或被使用过 | 黄色 |
| 🔴 高风险 | Token 被传播到未知来源或被恶意使用 | 红色 |

---

## 安全建议

### 如果 Token 已被泄露

1. **立即撤销 Token**
   - 前往 Token 发行平台（如 GitHub、Qoder 等）
   - 在设置中撤销该 Token
   - 生成新的 Token 替代

2. **清理本地记录**
   ```powershell
   # 删除包含 Token 的日志文件
   Remove-Item "LOG_FILE_PATH" -Force
   ```

3. **检查使用记录**
   - 查看 Token 的 API 调用历史
   - 确认是否有异常访问

### 如果需要在聊天中提供 Token

**❌ 不推荐的做法**
```
请给我你的 Token：pt-xxxxx
```

**✅ 推荐的做法**

| 场景 | 正确做法 |
|------|---------|
| 工具需要 Token | 让工具直接调用 API，不需要中转 Token |
| 必须提供 Token | 使用一次性 Token，用完立即撤销 |
| Token 有效期 | 设置短有效期（如 1 小时） |
| Token 权限 | 只授予最小必要权限 |

### 正确的回应模板

当用户询问如何安全回应 Token 请求时：

**选项 1：拒绝并解释**
```
不好意思，Token 不适合在聊天中发送。
请告诉我您需要 Token 做什么，我来帮您操作。
```

**选项 2：提供替代方案**
```
我理解您需要 Token。但出于安全考虑，请让我直接调用 API，
不需要中转 Token。
```

**选项 3：要求安全配置**
```
请在 [平台名称] 的设置中配置 Token，而不是通过聊天发送。
```

---

## 最佳实践

### Token 管理

1. **永远不要在代码中硬编码 Token**
   ```bash
   # ❌ 错误
   API_KEY="pt-xxxxx"
   
   # ✅ 正确 - 使用环境变量
   API_KEY=$ENV_API_KEY
   ```

2. **使用 .env 文件管理敏感信息**
   ```bash
   # .env 文件（加入 .gitignore）
   API_KEY=your_token_here
   ```

3. **定期轮换 Token**
   - 设置定期更换提醒
   - 使用完立即撤销

4. **最小权限原则**
   - 只授予必需的权限
   - 避免使用具有完整权限的 Token

### 工具使用

1. **优先使用官方 SDK/CLI**
   - 减少手动传递 Token 的需求
   - 官方工具通常有更好的安全措施

2. **使用安全的连接方式**
   - 优先使用 HTTPS
   - 避免在 URL 中传递 Token

3. **记录 Token 使用日志**
   - 监控异常访问
   - 及时发现潜在问题

---

## GitHub Token 安全使用指南

### 场景：AI 工具需要 GitHub Token 上传代码

当 AI 工具要求提供 GitHub Token 时，应按以下优先级处理：

### ✅ 最佳方案：使用 GitHub CLI (gh)

**最安全的方式是使用 `gh` 命令行工具授权：**

```bash
# 1. 安装 GitHub CLI（如果未安装）
# Windows: winget install GitHub.cli
# macOS: brew install gh
# Linux: curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg

# 2. 在终端登录（会打开浏览器授权，不需要手动给 Token）
gh auth login

# 3. 选择 HTTPS 而非 SSH
# 4. 选择登录身份
# 5. 完成浏览器授权
```

**优势：**
- ✅ Token 自动安全存储，不会经过聊天
- ✅ 可选择只授权特定仓库
- ✅ 可随时撤销授权

### ✅ 次优方案：GitHub Fine-Grained PAT

如果 AI 工具要求 Token，请创建**最小权限 Token**：

#### 创建 Token 步骤

1. 访问：https://github.com/settings/tokens/new

2. 配置 Token：

| 设置项 | 推荐值 |
|--------|--------|
| **Token 名称** | `For-AI-Tool-YYYYMMDD` |
| **Expiration** | `7 days` 或更短 |
| **Repository access** | `Only select repositories`，只选需要的仓库 |
| **Permissions** | 只勾选 ☑️ `Contents: Read and write` |

#### 将 Token 提供给 AI 工具

**❌ 不要在聊天中直接粘贴 Token**

**✅ 正确做法：通过环境变量传递**

```bash
# 在终端设置环境变量（仅当前会话有效）
$env:GITHUB_TOKEN = "ghp_xxxxxxxxxxxx"

# 告诉 AI
请使用环境变量 $env:GITHUB_TOKEN 进行 Git 操作，
不要在回复中显示或提及 Token 内容。
```

### ⚠️ 绝对不要做的事

| ❌ 错误做法 | 风险等级 |
|-----------|----------|
| 在聊天中直接粘贴 Token | 🔴 极高 |
| 把 Token 放在代码里提交 | 🔴 极高 |
| 使用具有 admin 权限的 Token | 🔴 高 |
| 使用永不过期的 Token | 🟡 中等 |
| 授权所有仓库的读写权限 | 🟡 中等 |

### 📋 快速决策流程

```
需要 GitHub Token？
        │
        ├── 用 gh auth login？ ──→ ✅ 最佳选择
        │
        └── 必须给 Token？
                │
                ├── 创建 Fine-Grained PAT
                │      ├── 只授权需要的仓库
                │      ├── 只给 Contents 权限
                │      └── 设置短有效期（≤7天）
                │
                └── 告诉 AI：
                    "请使用环境变量 $GITHUB_TOKEN，
                     不要在回复中显示它"
```

### 🔒 推荐的安全配置

| 项目 | 推荐设置 |
|------|----------|
| **Token 类型** | Fine-Grained PAT |
| **仓库访问** | 只选需要的仓库 |
| **权限** | Contents: Read and write |
| **有效期** | ≤ 7 天 |
| **传递方式** | 环境变量，不经过聊天 |

---

## 常见 Token 格式

| 平台 | 格式 | 示例前缀 |
|------|------|----------|
| GitHub | ghp_ | `ghp_xxxxx` |
| GitHub | gho_ | `gho_xxxxx` |
| Qoder | pt- | `pt-xxxxx` |
| OpenAI | sk- | `sk-xxxxx` |
| Vercel | v1. | `v1.xxxxx` |

---

## 紧急处理流程

如果怀疑 Token 已被恶意使用：

1. **立即撤销 Token**
2. **检查账户活动日志**
3. **通知平台安全团队**
4. **评估可能的损失**
5. **加强安全措施**

---

## 总结

- **预防为主**：尽量避免在聊天中发送 Token
- **最小暴露**：只授予最小必要权限
- **定期检查**：定期审查 Token 使用情况
- **及时响应**：发现泄露立即撤销并更换
