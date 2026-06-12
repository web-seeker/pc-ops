# 🧹 Windows 自带磁盘清理

> 不用装任何第三方软件。Windows 自带的 **cleanmgr.exe + Dism** 就是最安全、最高效的系统清理方案。

本模块逐项拆解每个清理选项，覆盖标准模式和管理员模式共 13 项，配完整 BAT/PowerShell 一键脚本。

---

## 为什么大多数人清完 C 盘还是红的

网上 99% 的教程只说"勾选全部 → 确定"。但 **左下角的「清理系统文件」按钮**才是关键——不点它，Windows 更新缓存（5~30 GB）和 Windows.old（10~30 GB）根本不会出现。这不是隐藏功能，是权限不足导致的"假清理"。

本篇不止告诉你「怎么做」，更拆解每项在磁盘的**真实路径**和**清理后果**，让你可以自信地勾选，而不是"全勾了听天由命"。

---

## 📦 内容快照

| 维度 | 覆盖内容 |
|------|---------|
| **GUI 拆解** | 标准模式 8 项 + 管理员模式 5 项，每项含磁盘路径、清理后果、安全评级 |
| **CLI 自动化** | cleanmgr 完整参数表（含 `/autoclean`）+ sageset/sagerun 注册表机制 |
| **Dism 引擎** | `StartComponentCleanup` vs `/ResetBase` 区别 + `AnalyzeComponentStore` 分析命令 |
| **开箱脚本** | 三套 BAT/PowerShell：标准版 → 静默版 → 深度版，复制即用 |
| **横向对比** | 为什么用自带工具而不是 CCleaner（含安全性 / 隐私 / 注册表风险说明） |
| **故障排查** | 5 个高频问题：卡住、空间不涨、Windows.old 消失、清理后变慢、WinSxS 能删吗 |

## 📄 文档

| 文档 | 说明 |
|------|------|
| [📘 完整操作指南](windows-disk-cleanup-skill.md) | 13 项逐条拆解 + 命令行 + 脚本 + FAQ，583 行 |

---

## 🚀 30 秒速成

```batch
Win + R → cleanmgr → 选 C 盘 → 确定
     ↓
点击左下角「清理系统文件」← 关键一步
     ↓
勾选所有 → 确定 → 删除文件
```

### 一键脚本

```batch
:: 管理员 CMD 中运行
cleanmgr /sagerun:1
Dism /online /Cleanup-Image /StartComponentCleanup /Quiet
```

> ⚠️ `/sagerun:1` 需先在管理员终端跑一次 `cleanmgr /sageset:1` 保存配置。

---

## 🛡️ 清理安全原则

```
✅ 放心清：回收站、临时文件、缩略图、错误报告、传递优化
⚠️ 注意：  Windows 更新清理（不可卸载）、驱动程序包（保留最新）
⚠️ 慎清：  以前的 Windows 安装（确认新系统稳定）、系统还原点（保留最近一个）
❌ 别干：  手动删 WinSxS 文件夹、用第三方工具清注册表
```

## 🔗 相关模块

- [🗂️ 磁盘分区合并](../disk-partition-merge/) — 如果清完空间还不够用
- [🔍 清除残留 SID](../sid-cleanup/) — 清理后权限排查
