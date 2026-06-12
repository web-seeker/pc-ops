---
name: token-security-audit
description: Token 安全检测与隐私保护。当用户提到 Token、API Key、密钥、密码、隐私、安全检测、token泄露检测、GitHub SSH、SSH认证、ClawHub、OAuth、skill发布时自动触发。用于检查 Token 是否泄露、安全使用最佳实践、隐私风险评估、SSH/OAuth 方案选择。
---

# Token 安全检测与隐私保护助手

当检测到 Token、API Key、密钥等敏感信息相关问题时，本技能将帮助用户：
- 检测 Token 是否已被记录或泄露
- 评估隐私风险等级
- 提供安全处理建议
- 指导正确的 Token / SSH 认证管理实践

---

## 使用场景

### 触发关键词
- "Token 安全吗？"
- "会泄露吗？"
- "怎么检测泄露？"
- "Token 注意事项"
- "ClawHub 发布"
- "skill 发布安全"
- "API Key 隐私"
- "密钥安全管理"
- "GitHub SSH"
- "不用 Token 怎么认证"

### 常见问题
1. 在聊天中发送了 Token，担心泄露
2. 工具要求提供 Token，不知道是否安全
3. 想检查本地是否有 Token 被记录
4. 不知道如何安全管理 Token
5. 想用 SSH 替代 Token 认证 GitHub

---

## 认证方案选择：SSH / OAuth vs Token

> **首选推荐：SSH 认证（零 Token 暴露）**
> Token 需要通过对话文本传递，有泄露风险；SSH 公钥可以公开，私钥永不离开本地。GitHub 官方推荐 SSH 作为首选认证方式。

### 各平台安全方案对比

| 维度 | GitHub SSH | ClawHub OAuth | 通用 Token |
|------|-----------|---------------|------------|
| 凭证传递 | 公钥可公开，私钥不出本地 | 浏览器授权，Token 存本地 | Token 需传递给 AI/工具 |
| 安全风险 | 极低 | 低（Token 不经过 AI） | 中高（可能泄露） |
| 官方推荐度 | 首选推荐 | 首选推荐 | 次选 / 兜底 |
| 适用场景 | Git 推送/拉取 | Skill 发布、平台 API | API 调用、精细化权限 |

### 👉 需要用 SSH？跳转到 [GitHub SSH 认证模块](#github-ssh-认证流程)
### 👉 需要用 OAuth 认证 ClawHub？跳转到 [ClawHub OAuth 认证模块](#clawhub-oauth-认证流程)

---

## 检测流程

### 第一步：检查本地日志/历史是否记录了 Token

```powershell
# PowerShell 检查常见日志位置
Get-ChildItem -Path "$env:USERPROFILE" -Recurse -Include "*.log","*.txt" -ErrorAction SilentlyContinue | 
    Select-String -Pattern "ghp_|gho_|sk-|pt-|v1\." -List | Select-Object Path,LineNumber
```

```bash
# Bash 检查 shell 历史
grep -l "ghp_|gho_|sk-|pt-|clawhub" ~/.bash_history ~/.zsh_history 2>/dev/null
```

### 第二步：检查平台授权记录

访问以下页面检查异常授权：
- https://github.com/settings/applications — 查看 OAuth 应用授权
- https://github.com/settings/tokens — 查看 Personal Access Token
- https://github.com/settings/ssh — 查看 SSH 公钥（推荐用这个代替 Token）
- https://clawhub.ai/settings/tokens — 查看 ClawHub API Token

### 第三步：检查环境变量

```powershell
# 检查环境变量中是否有 Token
Get-ChildItem env: | Where-Object { $_.Value -match "ghp_|sk-|pt-|claw" } | Select-Object Name,Value
```

---

## 风险等级评估

| 等级 | 说明 | 颜色 | 建议动作 |
|------|------|------|----------|
| 🟢 低风险 | Token 未被使用，仅出现在错误日志 | 绿色 | 清理日志，更换 Token |
| 🟡 中等风险 | Token 被记录在多个位置或被使用过 | 黄色 | 立即撤销，审查访问记录 |
| 🔴 高风险 | Token 被传播或出现异常 API 调用 | 红色 | 立即撤销+强制改密+通知平台 |

---

## 如果 Token 已被泄露：紧急处理

1. **立即撤销 Token** — 前往对应平台设置页撤销
2. **审查访问记录** — 查看 Token 的 API 调用历史，确认是否有异常
3. **清理本地记录** — 删除包含 Token 的日志/历史文件
4. **通知平台** — 如确认被恶意使用，联系平台安全团队
5. **加强防护** — 启用 2FA，更换为 SSH 认证

---

## GitHub Token 安全使用指南（当无法使用 SSH 时）

### ✅ 最佳方案：使用 GitHub CLI (`gh`)

```bash
# 浏览器授权，Token 不经过聊天
gh auth login
# 选择：GitHub.com → HTTPS → Login with a web browser
```

**优势：** Token 自动安全存储，可选择只授权特定仓库，可随时撤销。

### ✅ 次优方案：Fine-Grained PAT（最小权限）

| 设置项 | 推荐值 |
|--------|--------|
| Token 名称 | `For-AI-Tool-YYYYMMDD` |
| Expiration | `7 days` 或更短 |
| Repository access | `Only select repositories` |
| Permissions | 只勾选 `Contents: Read and write` |

**传递方式（❌ 不要粘贴到聊天）：**
```bash
# 在终端设置环境变量
$env:GITHUB_TOKEN = "ghp_xxxxxxxxxxxx"
# 然后告诉 AI：请用 $env:GITHUB_TOKEN，不要显示它
```

### ⚠️ 绝对不要做

| ❌ 错误做法 | 风险 |
|-----------|------|
| 在聊天中直接粘贴 Token | 🔴 极高 |
| Token 硬编码在代码里 | 🔴 极高 |
| 使用 admin 权限 Token | 🔴 高 |
| 永不过期的 Token | 🟡 中等 |

---

## 常见 Token 格式（识别用）

| 平台 | 格式前缀 |
|------|----------|
| GitHub PAT | `ghp_` |
| GitHub OAuth | `gho_` |
| OpenAI | `sk-` |
| Vercel | `v1.` |
| ClawHub API Token | `chub_` |

---

## 总结
- **首选 SSH / OAuth，次选 Token** — SSH 私钥不离开本地，OAuth 通过浏览器授权 Token 存本地，从根本上避免 Token 泄露
- **预防为主** — 尽量避免在聊天中发送 Token
- **最小暴露** — Token 只授予最小必要权限，短期有效
- **定期检查** — 定期审查各平台授权记录（GitHub + ClawHub）

---

## GitHub SSH 认证流程

---
name: github-ssh-auth
description: GitHub SSH 安全认证流程 — 零 Token 暴露。当需要授权 AI 访问 GitHub 时使用。
agent_created: true
---

# GitHub SSH 安全认证流程

## 为什么用 SSH 而不是 Token

- Token 需要通过对话文本传递，有泄露风险
- SSH 公钥可以公开，私钥永不离开本地
- GitHub 官方推荐 SSH 作为首选认证方式

## 标准流程

### 第一步：生成 SSH 密钥

```bash
# 检查是否已有密钥
ls ~/.ssh/id_ed25519.pub 2>/dev/null && echo "已存在" || echo "需生成"

# 如果没有，生成新密钥（ed25519 比 RSA 更快更安全）
ssh-keygen -t ed25519 -C "your-label" -f ~/.ssh/id_ed25519 -N ""

# 输出公钥（这串可以安全公开）
cat ~/.ssh/id_ed25519.pub
```

### 第二步：将公钥添加到 GitHub

1. 复制上一步输出的公钥（以 `ssh-ed25519` 开头的那一整行）
2. 打开 https://github.com/settings/ssh/new
3. Title：填任意标识（如 `my-pc`、`macbook`）
4. Key type：选择 `Authentication Key`
5. Key：粘贴公钥
6. 点击 "Add SSH key"

### 第三步：验证连接

```bash
# 添加 GitHub 到 known_hosts（首次连接必须）
ssh-keyscan github.com >> ~/.ssh/known_hosts

# 测试连接
# 成功应返回：Hi <username>! You've successfully authenticated...
ssh -T git@github.com
```

### 第四步：配置仓库使用 SSH

```bash
# 创建仓库时直接用 SSH 地址
gh repo create REPO_NAME --public --source . --remote origin --push

# 或手动设置已有仓库的远程地址
git remote set-url origin git@github.com:USERNAME/REPO.git
```

## 安全原则速查

| 可以公开 | 绝不能公开 |
|---------|-----------|
| 公钥 `.pub` 文件 | 私钥（无 `.pub` 后缀的文件） |
| 公钥内容可贴到任何对话 | 私钥内容绝不出本地 |
| GitHub 仓库 URL | 任何形式的 Token/密码 |

## 给其他 AI 的简洁指令模板

直接复制这段话发给任何 AI：

> 帮我用 SSH 方式连接 GitHub：
> 1. 生成 ed25519 密钥对（如果还没有）
> 2. 把公钥输出给我，我自己去 GitHub 设置页添加
> 3. 我添加完后告诉你，你验证连接
> 全程不用 Token，不暴露任何凭证。

## 常见问题

| 问题 | 解决 |
|------|------|
| `Host key verification failed` | 运行 `ssh-keyscan github.com >> ~/.ssh/known_hosts` |
| `Permission denied (publickey)` | 确认公钥已添加到 GitHub，检查 `ssh -T git@github.com` |
| 想换用 HTTPS 协议 | `git remote set-url origin https://github.com/USER/REPO.git` |
| 多 GitHub 账号管理 | 在 `~/.ssh/config` 配置不同 Host 别名 |
| ClawHub OAuth 失败 | 确认 CLI 已安装（`clawhub --version`），检查网络，重试 `clawhub login` |

---

## ClawHub OAuth 认证流程

---
name: clawhub-oauth-auth
description: ClawHub OAuth 安全认证流程 — 零 Token 暴露。当需要发布 Skill 到 ClawHub 或授权 AI 访问 ClawHub 时使用。
agent_created: true
---

# ClawHub OAuth 安全认证流程

## 为什么用 OAuth 而不是 --token

- `clawhub login --token <xxx>` 需要把 Token 明文写在命令行/对话中，有泄露风险
- `clawhub login`（无参数）走浏览器 OAuth，Token 由 CLI 本地存储，不经过对话
- OAuth 是 ClawHub 官方推荐且最安全的方式

## 标准流程

### 第一步：获取 ClawHub CLI

```bash
# 方式一：npm 全局安装（推荐）
npm install -g clawhub

# 方式二：下载 tgz 手动解压（Windows 权限有问题时用这个）
curl -L "https://registry.npmjs.org/clawhub/-/clawhub-0.20.0.tgz" -o clawhub.tgz
tar -xzf clawhub.tgz
# CLI 入口：package/bin/clawhub.js
# 运行方式：node package/bin/clawhub.js login
```

### 第二步：浏览器 OAuth 登录

```bash
# 直接运行 login，CLI 自动打开浏览器进行 GitHub OAuth
clawhub login

# 如果环境没有浏览器（headless），用 device flow
clawhub login --device
```

> 浏览器会打开 `https://clawhub.ai/cli/auth?...`，用 GitHub 账号授权即可。
> 授权后浏览器跳转回 `http://127.0.0.1:<port>/callback`，CLI 自动完成认证。
> **整个过程 Token 不在对话中出现。**

### 第三步：验证登录

```bash
clawhub whoami
# 应返回你的 ClawHub 账号名
```

### 第四步：发布 Skill

```bash
clawhub skill publish <skill-folder> \
  --slug <slug> \
  --name "Display Name" \
  --version 2.0.0 \
  --changelog "更新说明" \
  --tags "tag1,tag2,tag3"
```

## 注意事项

- ClawHub CLI 是 Node.js 包，需要 Node.js 环境
- 如果 npm 全局安装有问题（Windows 权限），可以手动下载 tgz 解压后用 `node bin/clawhub.js` 运行
- `NODE_OPTIONS` 环境变量可能干扰 CLI，遇到报错时 `NODE_OPTIONS=""` 再执行
- 同一个账号可以发布多个 skill，无需重新认证

## 安全原则速查

| 可以公开 | 绝不能公开 |
|---------|-----------|
| `clawhub login` 命令本身 | `clawhub login --token <value>` 中的 Token |
| 发布命令（不含 Token） | 任何以 `chub_` 开头的字符串 |
| ClawHub 账号名 | `.clawhub/` 目录下的 token 文件内容 |

## 给其他 AI 的简洁指令模板

直接复制这段话发给任何 AI：

> 帮我把 skill 发布到 ClawHub：
> 1. 安装 clawhub CLI（npm install -g clawhub）
> 2. 运行 clawhub login 走浏览器 OAuth 登录（不要用 --token）
> 3. 我授权完后告诉你
> 4. 然后发布 skill
> 全程走浏览器 OAuth，不让 Token 出现在对话中。

## 双平台同时发布指令模板

> 帮我把这个 skill 同时发布到 GitHub 和 ClawHub：
> - GitHub 用 SSH 认证，公钥输出给我去添加
> - ClawHub 用浏览器 OAuth，我手动授权
> - 全程不暴露任何 Token 或密码在对话中

## 常见问题

| 问题 | 解决 |
|------|------|
| `clawhub: command not found` | 用 tgz 方式安装，或检查 npm global bin 路径 |
| OAuth 页面打不开 / 无浏览器 | 用 `clawhub login --device` device flow |
| EACCES / Windows 权限错误 | 下载 tgz 手动解压，用 `node bin/clawhub.js` 运行 |
| `NODE_OPTIONS` 报错 | 运行前执行 `NODE_OPTIONS=""` 清除环境变量 |
| 发布失败权限不足 | 确认 `clawhub whoami` 返回的账号有发布权限 |
