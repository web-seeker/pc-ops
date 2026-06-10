<p align="center">
  <img src="https://img.shields.io/badge/platform-Windows-blue?style=flat-square" alt="Windows">
  <img src="https://img.shields.io/badge/tools-PowerShell%20%2B%20AOMEI%20%2B%20Git%20SSH-green?style=flat-square" alt="Tools">
  <img src="https://img.shields.io/badge/license-MIT-lightgrey?style=flat-square" alt="MIT">
  <img src="https://img.shields.io/badge/verified-2026.06-success?style=flat-square" alt="Verified">
</p>

# pc-ops 🖥️

**个人电脑运维实战手册 —— 用 AI 桌面助手辅助搞定磁盘、权限、安全认证、系统配置等日常操作。**

每项技能独立成章，图文并茂，含完整命令和踩坑记录。不止告诉你「怎么做」，更会告诉你「AI 在哪些地方栽过跟头、为什么」。

---

## 📦 技能模块

### 🗂️ 磁盘分区合并

> 删除废弃分区 → 释放 173GB → 合并到 C 盘 → 恢复 BitLocker 加密

```
扩容前  [C: 195GB ████████░░░░░░░░]
扩容后  [C: 368GB ████████████████░]
```

| 文档 | 说明 |
|------|------|
| [📘 全流程操作指南](disk-partition-merge/F盘空间合并至C盘操作全流程指南.md) | 图文并茂，每步有截图。含免责声明 |
| [📄 技能文档](disk-partition-merge/disk-partition-merge-bitlocker-skill.md) | 技术向全记录，完整 PowerShell 命令与故障排查 |

包含 5 张 AOMEI 操作截图（主界面、右键菜单、套用按钮、执行确认、语言设置）。

> 💡 AI 负责命令行（diskpart / manage-bde），你花 3 分钟手动拖拽分区。AOMEI 的 MFC 界面是 AI 自动化的黑洞。

---

### 🔐 安全认证：Token & SSH

> **Token 泄露检测 + SSH/OAuth 安全认证** — 从「怎么安全用 Token」到「SSH/OAuth 彻底不用 Token」

本模块覆盖两类核心场景：

#### 场景 A：必须用 Token 时

当 AI 工具要求提供 GitHub Token 时：

| 方案优先级 | 推荐方式 |
|-----------|---------|
| ✅ 最佳 | `gh auth login` — 浏览器授权，Token 不经过聊天 |
| ✅ 次优 | Fine-Grained PAT — 最小权限 + ≤7天有效期 + 环境变量传递 |

**绝对不要做的事：**
- ❌ 在聊天中直接粘贴 Token
- ❌ Token 硬编码在代码里提交
- ❌ 使用 admin 权限或永不过期的 Token

#### 场景 B：🔥 推荐 — 用 SSH 替代 Token（零暴露）

> **为什么？** Token 需要通过对话文本传递，有泄露风险。SSH 公钥可以公开，私钥永不离开本地。GitHub 官方推荐 SSH 为首选认证方式。

```
生成密钥对 → 输出公钥(安全公开) → 用户自行添加到 GitHub → 验证连接
全程零 Token，凭证不离开本地机器
```

| 文档 | 说明 |
|------|------|
| [📄 安全认证 Skill](token-security/SKILL.md) | Token 泄露检测 + SSH/OAuth 三模块，含方案对比表、可执行的检测命令、SSH 四步标准流程 |

| 维度 | Token（Fine-Grained PAT） | GitHub SSH | ClawHub OAuth |
|------|--------------------------|-----------|-------------|
| 凭证传递 | 通过环境变量给 AI | 私钥不离开本地 | 浏览器授权，Token 存本地 |
| 安全风险 | 中高 | 极低 | 低 |
| 适用场景 | API 调用、精细化权限 | **Git 操作首选** | **Skill 发布首选** |

> 💡 **使用顺序：GitHub 用 SSH，ClawHub 用 OAuth → 无法用时再用 Token**。Skill 文档内有完整的「给其他 AI 的简洁指令模板」，直接复制就能让任何 AI 帮你走 SSH 流程。

---

### 🔍 清除残留 SID 账户

> 文件夹 → 属性 → 安全 → 发现一个「未知账户(S-1-5-21-xxx)」删不掉？

这是已删除用户的残留 SID（Security Identifier）。虽然一般不影响使用，但权限列表看着碍眼，安全审计工具也会标记。本模块记录了从识别到彻底清除的全过程。

**实际操作案例**：在 `C:\ProgramData\Microsoft\Windows\Start Menu` 中发现 `S-1-5-21-1482809247-1092165676-3601881385-1000`，经全盘扫描确认仅此一处，已成功移除。

**四步流程**：

```
识别目标 SID → 全面扫描（文件系统 + 注册表 + 任务计划 + 服务）→ 逐项清除 → 验证无残留
```

**扫描范围覆盖**：

| 检查项 | 方法 |
|--------|------|
| 文件系统 ACL | `icacls /findsid` + `Get-Acl` 递归扫描 ProgramData、Users、Windows |
| 注册表 | ProfileList、ProfileGuid、AppID |
| 计划任务 | `Get-ScheduledTask` 过滤 Principal.UserId |
| Windows 服务 | WMI 查询 StartName |
| SID 命名文件夹 | `C:\Users\S-1-5-21-*` 残留目录 |

| 文档 | 说明 |
|------|------|
| [📘 操作指南](sid-cleanup/清除残留SID账户操作指南.md) | 完整操作流程：识别 → 扫描 → 清除 → 验证。含全部 PowerShell 命令和截图 |

包含权限条目示例截图（其他账户已打码处理）。

> 💡 手动翻属性窗口删 ACL 条目要十几分钟，AI 写个 `Get-Acl` + `Set-Acl` 脚本三分钟扫完整个 ProgramData 并清理干净。这类重复性权限操作最适合交给 AI。

---

## 📁 仓库结构

```
pc-ops/
├── README.md
│
├── disk-partition-merge/
│   ├── F盘空间合并至C盘操作全流程指南.md
│   ├── disk-partition-merge-bitlocker-skill.md
│   └── aomei-*.png (5张截图)
│
├── sid-cleanup/
│   ├── 清除残留SID账户操作指南.md
│   └── sid-permission-example.png
│
└── token-security/
    └── SKILL.md          ← 双模块：Token安全检测 + SSH认证流程
```

---

## 🚀 快速开始

```bash
git clone https://github.com/web-seeker/pc-ops.git
```

进入对应模块文件夹，打开 `.md` 文档即可。每个模块独立自包含。

> ⚠️ 操作前务必完整阅读文档中的免责声明和注意事项。磁盘和权限操作有不可逆风险，请确认数据已备份。

---

## 📝 License

MIT · 详见各文档内免责声明

---

<p align="center">
  <sub>🤖 由 WorkBuddy 辅助生成 · 所有步骤已实际操作验证 · 2026.06</sub>
</p>
