# pc-ops

> 个人电脑操作实战手册 —— 用 AI 辅助完成磁盘分区、系统配置等日常运维任务。

## 关于本仓库

本仓库记录了使用 **QoderWork CN（User 版）** 桌面 AI 助手辅助完成 Windows 系统运维的完整流程。每份技能文档包含命令行操作、GUI 操作截图、注意事项和踩坑记录，既可作为日后参考手册，也可分享给遇到类似问题的用户。

AI 不是万能的 —— 本仓库最大价值在于明确划分了 **AI 能做** 和 **必须手动** 的边界，避免你在自动化操作中反复踩坑。

## 仓库结构

```
pc-ops/
├── disk-partition-merge-bitlocker-skill.md  # 磁盘分区合并 + BitLocker 恢复 完整技能文档
├── 分区操作指南.md                            # 简版分区操作指南
├── aomei-main-interface.png                 # AOMEI 主界面截图（操作完成后）
├── aomei-resize-move.png                    # 右键菜单「调整/移动分区」
├── aomei-apply-button.png                   # 顶部「套用」按钮
├── aomei-apply-restart.png                  # 执行确认 + 重启提示
├── aomei-language-menu.png                  # 语言设置路径
└── README.md                                # 本文件
```

## 已收录内容

### Windows 磁盘分区合并 + BitLocker 恢复

**场景**：删除废弃分区 → 将释放空间合并到 C 盘 → 操作后恢复 BitLocker 加密。

**实际操作案例**：删除 F 盘（172GB）→ C 盘从 195GB 扩容至 368GB → 重新启用 BitLocker 保护。

**涉及工具**：

| 工具 | 用途 |
|------|------|
| QoderWork CN（User 版） | AI 桌面助手，负责命令行操作与流程编排 |
| AOMEI Partition Assistant 标准版 | 第三方分区工具，手动拖拽移动/扩展分区 |
| Windows diskpart | 删除分区 |
| Windows manage-bde | BitLocker 加密管理 |

**核心经验**：

- AOMEI 的 MFC 图形界面 AI 无法可靠自动化（管理员权限隔离、DPI 缩放、自定义控件不暴露给无障碍 API）。AI 尝试自动化时反复崩溃。结论：**GUI 拖拽操作必须手动完成**，AI 负责命令行部分即可
- Windows 原生磁盘管理不支持移动分区，必须借助第三方工具
- BitLocker 生命周期：操作分区前暂停保护 → 重启后重新启用 → 为数据盘开启自动解锁

更多细节见 [完整技能文档](disk-partition-merge-bitlocker-skill.md)。

## 适用人群

- Windows 用户，有一定基础，愿意尝试用 AI 辅助运维
- 遇到过类似磁盘分区问题的人，可直接参考操作流程
- 对 QoderWork 桌面 AI 助手实际能力边界感兴趣的用户

## 使用方式

1. 克隆仓库到本地：
   ```bash
   git clone https://github.com/web-seeker/pc-ops.git
   ```

2. 根据需求打开对应的 `.md` 技能文档

3. ⚠️ 执行任何磁盘操作前，务必确认数据已备份，并仔细阅读文档中的「注意事项」部分

## 许可

MIT License

---

🤖 部分内容由 [QoderWork](https://qoder.com) 辅助生成 · 所有操作流程均经实际验证
