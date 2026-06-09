<p align="center">
  <img src="https://img.shields.io/badge/platform-Windows-blue?style=flat-square" alt="Windows">
  <img src="https://img.shields.io/badge/tools-AOMEI%20%2B%20diskpart-green?style=flat-square" alt="Tools">
  <img src="https://img.shields.io/badge/license-MIT-lightgrey?style=flat-square" alt="MIT">
  <img src="https://img.shields.io/badge/verified-2026.06-success?style=flat-square" alt="Verified">
</p>

# pc-ops 🖥️

**C 盘红了？删了个废弃分区不知道怎么合并回去？BitLocker 锁着不敢动？**

这个仓库记录了用 **AI 桌面助手 + 第三方分区工具** 搞定 Windows 磁盘扩容的全过程——从 195GB 吃到 368GB，每一步都有截图、命令和踩坑说明。不止告诉你「怎么做」，还会告诉你「AI 在这里栽过什么跟头」。

---

## 📊 实际战果

```
扩容前  [C: 195GB ████████░░░░░░░░] 天天弹窗「磁盘空间不足」
扩容后  [C: 368GB ████████████████░] 半年内不用再清理
```

一次操作，省下以后每个月清理磁盘的焦虑时间。

---

## 🎯 这里有什么

| 文档 | 内容 |
|------|------|
| **[📘 F盘空间合并至C盘操作全流程指南](F盘空间合并至C盘操作全流程指南.md)** | 主文档——图文并茂的完整操作流程，从检查磁盘到 BitLocker 恢复，每一步都有截图 |
| **[📄 技能文档](disk-partition-merge-bitlocker-skill.md)** | 技术向全记录——完整的 PowerShell 命令、AOMEI 操作细节、故障排查手册 |

附带 5 张截图，覆盖所有关键操作节点：右键菜单、拖拽分区、套用按钮、执行确认、语言设置。

---

## ⚡ 三句话说完

1. **AI 用 diskpart 删掉 F 盘** → 释放 173GB 未分配空间
2. **你手动在 AOMEI 里把 E → D → C 依次拖拽** → 3 分钟搞定
3. **点击套用，重启** → PreOS 模式自动执行 10~30 分钟，C 盘扩容完成

> 💡 **为什么第二步必须手动？** 我们试过 pywinauto、SendInput、UIAutomation、屏幕坐标计算……全部失败。AOMEI 的 MFC 界面是 AI 自动化的黑洞。花 3 分钟自己拖拽，比 AI 在里面无限死循环高效 100 倍。

---

## 🛠️ 用到的工具

| 工具 | 做什么 | 谁操作 |
|------|--------|--------|
| QoderWork CN | 检查磁盘、运行 diskpart、管理 BitLocker | 🤖 AI |
| AOMEI Partition Assistant | 拖拽移动分区、扩展 C 盘 | 👤 你 |
| diskpart | 删除目标分区 | 🤖 AI |
| manage-bde | 暂停/恢复 BitLocker 加密 | 🤖 AI |

---

## 🧠 核心洞察

**AI 和 GUI 操作的边界划分，是本仓库最有价值的部分。**

Windows 原生磁盘管理不支持移动分区 → 必须用 AOMEI → AOMEI 的 MFC 控件不暴露给无障碍 API → AI 无法可靠点击右键菜单 → 但 AI 擅长跑命令、查状态、写脚本。所以正确的分工是：

```
AI 负责 CLI（命令行）→ 你负责 GUI（拖拽）→ AI 接手后续（BitLocker 恢复）
```

这层边界一旦划清，整个流程顺畅如水。划不清，就会像我们最初那样，在自动化 GUI 的路上反复撞墙。

---

## 🚀 快速开始

```bash
git clone https://github.com/web-seeker/pc-ops.git
```

打开 **[📘 F盘空间合并至C盘操作全流程指南](F盘空间合并至C盘操作全流程指南.md)** 开始阅读。

> ⚠️ **操作前务必完整阅读文档中的免责声明和注意事项。磁盘操作有不可逆风险，请确认数据已备份。**

---

## 📁 文件清单

```
pc-ops/
├── F盘空间合并至C盘操作全流程指南.md               # 👈 先看这个
├── disk-partition-merge-bitlocker-skill.md  # 完整技能文档（技术向）
├── aomei-main-interface.png                 # 操作完成后的分区布局
├── aomei-resize-move.png                    # 右键菜单「调整/移动分区」
├── aomei-apply-button.png                   # 「套用」按钮位置
├── aomei-apply-restart.png                  # 执行确认与重启提示
├── aomei-language-menu.png                  # 语言设置路径
└── README.md
```

---

## 📝 License

MIT · 随便用，后果自负（详见文档内免责声明）

---

<p align="center">
  <sub>🤖 由 QoderWork 辅助生成 · 所有步骤已实际操作验证 · 2026.06</sub>
</p>
