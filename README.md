<p align="center">
  <img src="https://img.shields.io/badge/platform-Windows-blue?style=flat-square" alt="Windows">
  <img src="https://img.shields.io/badge/tools-PowerShell%20%7C%20AOMEI%20%7C%20cleanmgr%20%7C%20SSH-green?style=flat-square" alt="Tools">
  <img src="https://img.shields.io/badge/license-MIT-lightgrey?style=flat-square" alt="MIT">
  <img src="https://img.shields.io/badge/verified-2026.06-success?style=flat-square" alt="Verified">
</p>

# pc-ops 🖥️

> **个人电脑运维实战手册 — 磁盘、安全、权限、清理，每项一个独立模块，拿来即用。**

---

## 📦 四大模块

```
         你的电脑
            │
   ┌────────┼────────┬────────┐
   ▼        ▼        ▼        ▼
 磁盘空间  安全认证  系统清理  权限审计
   │        │        │        │
   ▼        ▼        ▼        ▼
 分区合并  Token/SSH  磁盘清理  SID清理
 (物理)    (配置)    (软件)    (审计)
```

---

## 🧹 Windows 磁盘清理（高亮推荐 ⭐）

> **C 盘红了但不想动分区？用系统自带工具，安全清出 5~30 GB。**

### 为什么大多数人清完 C 盘还是红的？

网上 99% 的教程只说「勾选全部 → 确定」。但**「清理系统文件」按钮才是关键**——不点它，Windows 更新缓存（5~30 GB）和 Windows.old（10~30 GB）根本不会出现。

### 操作流程截图

**Step 1：打开磁盘清理，选 C 盘**

![cleanmgr 主界面](windows-disk-cleanup/1.png)

```
Win + R → cleanmgr → 选 C 盘 → 确定
```
此时看到的是**标准模式**，只有回收站、临时文件等零头。

**Step 2：点击「清理系统文件」← 关键一步！**

![清理系统文件按钮](windows-disk-cleanup/2.png)

点击后需要管理员权限（UAC 确认），重新扫描后会出现大项。

**Step 3：系统文件清理项出现**

![系统文件清理列表](windows-disk-cleanup/3.png)

此时可见 **Windows 更新清理（5~30 GB）**、**以前的 Windows 安装（10~30 GB）** 等大头。

**Step 4：勾选 + 确定，开始清理**

![清理进度中](windows-disk-cleanup/4.png)

Dism 后台还会清理 WinSxS 旧组件。

**Step 5：一键脚本（可选）**

![命令行一键清理](windows-disk-cleanup/5.png)

```batch
:: 管理员 CMD 中运行
cleanmgr /sagerun:1
Dism /online /Cleanup-Image /StartComponentCleanup /Quiet
```

> ⚠️ `/sagerun:1` 需要先跑一次 `cleanmgr /sageset:1` 保存配置。

### 安全原则速查

```
✅ 放心清   回收站、临时文件、缩略图、错误报告、传递优化
⚠️ 注意     Windows 更新清理（不可卸载更新）、驱动程序包（保留最新）
⚠️ 慎清      以前的 Windows 安装（确认新系统稳定）、系统还原点（保留最近一个）
❌ 别干      手动删 WinSxS 文件夹、用第三方工具清注册表
```

| 文档 | 内容 |
|------|------|
| [📘 完整操作指南](windows-disk-cleanup/windows-disk-cleanup-skill.md) | 13 项逐条拆解 + CLI 自动化 + 三套脚本 + FAQ |

---

## 🗂️ 磁盘分区合并

> **C 盘满了，旁边有个废盘 — 物理扩容，一劳永逸。**

| 维度 | 说明 |
|------|------|
| **解决问题** | C 盘空间不足，有废弃分区可删除，释放空间合并到 C 盘 |
| **涉及什么** | 删除分区 → 拖拽移动分区 → 扩展 C 盘 → 恢复 BitLocker 加密 |
| **风险等级** | 🔴 高 — 分区操作不可逆，数据丢失不可恢复 |
| **工具** | AOMEI Partition Assistant（免费）、Windows diskpart / manage-bde |

```
扩容前  [C: 195GB ████████░░░░░░░░]
扩容后  [C: 368GB ████████████████░]
```

**一句话**：这是物理扩容——真的动了磁盘分区表。和上面磁盘清理是互补关系：先清理，不够再扩容。

![AOMEI 操作截图](disk-partition-merge/aomei-main-interface.png)

| 文档 | 内容 |
|------|------|
| [📘 全流程操作指南](disk-partition-merge/F盘空间合并至C盘操作全流程指南.md) | 每步带截图 |
| [📄 技能参考](disk-partition-merge/disk-partition-merge-bitlocker-skill.md) | PowerShell 命令 + 故障排查 |

---

## 🔐 Token & SSH 安全认证

> **在聊天框里粘贴 Token？等你看到这条的时候，它可能已经泄露了。**

| 方案 | 凭证经过对话文本？ | 安全等级 | 适用场景 |
|------|:---:|:---:|------|
| GitHub SSH | ❌ 不经过 | 🟢 极低风险 | Git push/pull（首选） |
| ClawHub OAuth | ❌ 不经过 | 🟢 低风险 | Skill 发布（首选） |
| Fine-Grained PAT | ⚠️ 经过但可限制 | 🟡 中风险 | API 调用、兜底方案 |

> **核心原则**：Token 要经过对话文本才能给 AI = 不安全。SSH 私钥不出本地、OAuth 走浏览器授权 = Token 从不出现。后者永远优先。

| 文档 | 内容 |
|------|------|
| [📄 Token & SSH 安全认证](token-security/token-ssh-auth.md) | 泄露检测 + SSH 四步 + OAuth 三步 + AI 指令模板 |

---

## 🔍 清除残留 SID 账户

> **文件夹属性里有个「未知账户 (S-1-5-21-xxx)」删不掉？系统重装留下的孤儿 SID。**

```
识别 SID → 全盘扫描（文件系统+注册表+任务计划+服务） → 逐项清除 → 验证
```

![SID 权限示例](sid-cleanup/sid-permission-example.png)

| 文档 | 内容 |
|------|------|
| [📘 操作指南](sid-cleanup/清除残留SID账户操作指南.md) | PowerShell 命令 + 实战案例 |

---

## 🔀 什么时候用哪个？

```
C 盘红了 ──┬── 先跑「磁盘清理」清缓存 → 还是不够 → 再跑「分区合并」物理扩容
           └── 两者互补，不是二选一

安全认证 ──── 独立模块。跟磁盘没关系，但每个用 AI 写代码的人都该配置。

SID 残留 ──── 独立模块。系统重装后按需运行，跟上面三个都没关系。
```

---

## 📁 仓库结构

```
pc-ops/
├── README.md
├── windows-disk-cleanup/          ← 软件清理
│   ├── README.md
│   ├── windows-disk-cleanup-skill.md
│   └── 1-5.png (操作截图)
├── disk-partition-merge/          ← 物理扩容
│   ├── README.md
│   ├── F盘空间合并至C盘操作全流程指南.md
│   ├── disk-partition-merge-bitlocker-skill.md
│   └── aomei-*.png (截图)
├── token-security/                ← 认证安全
│   └── token-ssh-auth.md
└── sid-cleanup/                   ← 权限审计
    ├── README.md
    ├── 清除残留SID账户操作指南.md
    └── sid-permission-example.png
```

---

## 🚀 快速开始

```bash
git clone https://github.com/web-seeker/pc-ops.git
```

**建议顺序**：磁盘清理 →（不够？）→ 分区合并 → 顺手配 SSH → 哪天发现「未知账户」→ SID 清理

> ⚠️ 磁盘和权限操作有不可逆风险。操作前完整阅读对应模块的免责声明。

---

## 📄 License

MIT · 所有步骤已实际操作验证 · 2026.06

<p align="center">
  <sub>Made with ❤️ by <a href="https://github.com/web-seeker">web-seeker</a></sub>
</p>
