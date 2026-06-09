<p align="center">
  <img src="https://img.shields.io/badge/platform-Windows-blue?style=flat-square" alt="Windows">
  <img src="https://img.shields.io/badge/tools-PowerShell%20%2B%20AOMEI-green?style=flat-square" alt="Tools">
  <img src="https://img.shields.io/badge/license-MIT-lightgrey?style=flat-square" alt="MIT">
  <img src="https://img.shields.io/badge/verified-2026.06-success?style=flat-square" alt="Verified">
</p>

# pc-ops 🖥️

**个人电脑运维实战手册 —— 用 AI 桌面助手辅助搞定磁盘、权限、系统配置等日常操作。**

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

### 🔍 清除残留 SID 账户

> 文件权限中出现「未知账户(S-1-5-21-xxx)」→ 全盘扫描 → 逐项清除 → 验证无残留

| 文档 | 说明 |
|------|------|
| [📘 操作指南](sid-cleanup/清除残留SID账户操作指南.md) | 覆盖 icacls、PowerShell ACL、注册表、任务计划、服务账户 |

包含权限条目示例截图（其他账户已打码）。

> 💡 `Get-Acl` + `Set-Acl` 比手动翻 GUI 属性窗口快十倍。AI 写脚本三分钟扫完整个 ProgramData。

---

### 📦 模块扩展

每个技能模块独立一个文件夹，文档和截图自包含。新增技能只需：

1. 新建一个文件夹（文件夹名即技能标识）
2. 放入文档和截图
3. 在下方添加一个模块卡片

```
pc-ops/
├── disk-partition-merge/     ← 示例：磁盘分区合并
├── sid-cleanup/              ← 示例：清除残留 SID
├── your-new-skill/           ← 你的新技能
└── README.md
```

---

## 📁 仓库结构

```
pc-ops/
├── README.md
│
├── disk-partition-merge/
│   ├── F盘空间合并至C盘操作全流程指南.md
│   ├── disk-partition-merge-bitlocker-skill.md
│   ├── aomei-main-interface.png
│   ├── aomei-resize-move.png
│   ├── aomei-apply-button.png
│   ├── aomei-apply-restart.png
│   └── aomei-language-menu.png
│
└── sid-cleanup/
    ├── 清除残留SID账户操作指南.md
    └── sid-permission-example.png
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
  <sub>🤖 由 QoderWork 辅助生成 · 所有步骤已实际操作验证 · 2026.06</sub>
</p>
