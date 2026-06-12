# Windows 磁盘分区合并 + BitLocker 恢复完整流程

> 适用场景：删除多余分区，将释放空间合并到 C 盘，并在操作后恢复 BitLocker 加密。
>
> 本次实际操作：**删除 F 盘（172GB WORK 分区）→ C 盘扩容（195GB → 368GB）→ 恢复 BitLocker 加密**。

## 使用工具

| 工具 | 说明 |
|------|------|
| AI 桌面助手 | 负责命令行操作（磁盘检查、diskpart 删除分区、BitLocker 管理） |
| **AOMEI Partition Assistant Standard Edition**（傲梅分区助手 标准版） | 第三方磁盘分区工具，用于移动分区和扩展 C 盘。免费版即可完成操作。下载地址：https://www.disktool.cn/download.html |
| **Windows diskpart** | Windows 自带磁盘管理命令行工具，用于删除 F 盘分区 |
| **Windows manage-bde** | Windows 自带 BitLocker 管理命令行工具，用于暂停和恢复磁盘加密保护 |

---

## 前置准备

### 1. 确认磁盘布局

```powershell
# 以管理员权限运行 PowerShell，查看当前分区
Get-Partition -DiskNumber 0 | Format-Table PartitionNumber, DriveLetter, @{N="SizeGB";E={[math]::Round($_.Size/1GB,1)}}, Type
```

### 2. 确认目标分区数据情况

删除分区前，确认该分区没有重要数据，或已备份。

```powershell
Get-Volume -DriveLetter C, D, E, F | Format-Table DriveLetter, @{N="SizeGB";E={[math]::Round($_.Size/1GB,1)}}, @{N="FreeGB";E={[math]::Round($_.SizeRemaining/1GB,1)}}
```

### 3. 检查 BitLocker 状态（重要！）

在操作分区前，需要先解密/暂停受 BitLocker 保护的驱动器：

```powershell
# 以管理员权限查看 BitLocker 状态
manage-bde -status
```

如果某盘显示「保护状态: 已启用」，需要先暂停保护：

```powershell
manage-bde -protectors -disable C:
manage-bde -protectors -disable D:
manage-bde -protectors -disable E:
```

> **警告**：不暂停 BitLocker 保护就操作分区，可能导致数据无法访问。

---

## 第一步：删除目标分区（释放空间）

> **🚨 危险操作警告 —— 执行前必须逐条确认以下全部条件：**
>
> 1. 目标分区（如 F 盘）**数据为空或已全部备份**，删除后数据不可恢复
> 2. 用户已**反复确认**该分区没有需要保留的文件
> 3. 分区号已通过 `list partition` 确认无误
> 4. 确认删除的是正确的分区（不是系统盘、不是数据盘）
>
> **不允许 AI 在未获得用户明确确认的情况下直接执行删除操作。** 本次实际操作中，先通过 PowerShell 检查了 F 盘的已用空间为 0，再由用户口头确认「没有重要数据」后才执行。

使用 diskpart 删除分区，空间变为「未分配」。

```powershell
# 创建 diskpart 脚本（替换 <目标分区号>）
@"
select disk 0
list partition
select partition <目标分区号>
delete partition override
"@ | Out-File -FilePath delete_partition.txt -Encoding ASCII

# 以管理员权限执行
Start-Process diskpart -ArgumentList "/s $PWD\delete_partition.txt" -Verb RunAs -Wait
```

> `<目标分区号>` 用 `list partition` 确认后替换。

---

## 第二步：安装分区工具

**Windows 原生工具（磁盘管理、diskpart）无法移动分区**，需要 AOMEI Partition Assistant。

- **下载**：https://www.disktool.cn/download.html
- **版本**：标准版（免费）
- **安装后以管理员权限运行**

### 语言设置

AOMEI 安装后默认界面为繁体中文。如果偏好简体中文，可通过以下路径切换：

1. 点击右上角 **三条横线菜单（☰）**
2. 选择「**設定**」（设置）图标
3. 在语言选项中查找中文

![AOMEI 语言设置菜单](aomei-language-menu.png)

> **注意**：目前 AOMEI 标准版仅提供**繁体中文**，不提供简体中文选项。繁体中文界面与简体中文用词略有差异（如「套用」= 应用、「調整」= 调整、「分割區」= 分区），但操作流程完全一致，不影响使用。

> **⚠️ 重要：安装工具后，接下来的分区移动操作请手动完成，不要让 AI 自动化操作。**
>
> 原因：AOMEI 的图形界面基于 MFC 定制控件开发，存在以下自动化障碍——
> - 控件不向 UIAutomation（无障碍 API）暴露内部状态，AI 无法读取分区名称和菜单项
> - 管理员权限隔离导致鼠标模拟 API（SetCursorPos / SendInput）频繁失败
> - DPI 缩放（如 200%）导致屏幕坐标计算反复偏差
> - 右键菜单为 MFC 内嵌绘制，不是标准 Windows 弹出窗口，无法被程序检测
>
> **本次实际操作中，AI 在尝试自动化移动 E 盘时反复陷入崩溃和死循环。** 因此强烈建议：AI 负责命令行部分（检查磁盘、删除分区、恢复 BitLocker），**手动操作部分（AOMEI 拖拽移动分区）由用户亲自完成**。操作流程清晰简单，只需右键 → 拖拽 → 确定三步即可。

---

## 第三步：AOMEI 手动操作 —— 移动分区 + 扩展 C 盘

### 操作前布局

```
[System 260MB] [MSR 128MB] [C: 195GB] [D: 408GB] [E: 176GB] [未分配 ~173GB] [Recovery 1GB]
```

目标：将未分配空间合并到 C 盘。需要依次把 E 盘和 D 盘向右推。

### 操作完成后效果

![AOMEI 主界面 - 分区布局](aomei-main-interface.png)

> 上图是操作完成后的界面：C 盘已扩容至 **368.17GB**，D 盘 408.41GB，E 盘 175.91GB。每盘上的 🔒 图标表示 BitLocker 加密。

---

### 3.1 移动 E 盘到最右侧

1. **右键点击 E 盘**（PerFiles）
2. 在弹出的菜单中选择「**调整/移动分区**」

![右键菜单 - 调整/移动分区](aomei-resize-move.png)

3. 在弹出的对话框中，用鼠标**按住 E 盘的分区块，拖到最右侧**（紧挨 Recovery 分区）
4. 点击「确定」

> 此时未分配空间从 E 盘右侧移到了 E 盘左侧（D 盘右侧）。

### 3.2 移动 D 盘到右侧

1. 右键点击 D 盘（Data）→ 选择「调整/移动分区」
2. 将 D 盘分区块**拖到最右侧**（紧挨 E 盘）
3. 点击「确定」

> 此时未分配空间移到了 D 盘左侧，也就是 **C 盘正右侧**，C 盘与未分配空间相邻了。

### 3.3 扩展 C 盘

1. 右键点击 C 盘 → 选择「调整/移动分区」
2. 将 C 盘**右侧边界向右拖动**，纳入全部未分配空间
3. 点击「确定」

> C 盘从 195GB 扩展到约 **368GB**。

---

### 3.4 ⚠️ 点击「套用」—— 最关键的一步！

**前面的操作只是「排队」，还没有实际修改磁盘！**

在 AOMEI 顶部工具栏，**点击「套用」按钮**（绿色勾号 ✓）：

![AOMEI 套用按钮](aomei-apply-button.png)

> **只有点击「套用」，分区操作才会被确认并进入执行阶段。**

### 3.5 确认执行 + 重启电脑

点击「套用」后弹出确认对话框，检查操作列表无误：

- 移动 E 盘（向右）
- 移动 D 盘（向右）
- 调整 C 盘（扩展）

点击「执行」后，系统提示需要重启：

![AOMEI 执行对话框](aomei-apply-restart.png)

**确认重启**。电脑重启后，AOMEI 在 PreOS 模式（Windows 启动前）自动执行操作。屏幕会显示进度条，整个过程 **10-30 分钟**。

> **「套用」→「执行」→「重启」三步缺一不可。只拖拽不点套用 = 什么都没做。**

---

### 操作流程一览

```
右键 E → 调整/移动 → 拖到最右 → 确定
    ↓
右键 D → 调整/移动 → 拖到最右 → 确定
    ↓
右键 C → 调整/移动 → 扩展右边界 → 确定
    ↓
点击顶部「套用」按钮 ✓
    ↓
确认 → 执行 → 重启电脑
    ↓
PreOS 自动执行（10-30分钟）→ 完成
```

---

## 第四步：恢复 BitLocker 加密

分区操作完成后进入 Windows，重新启用 BitLocker 保护。

### 4.1 检查状态

```powershell
manage-bde -status
```

期望看到「保护状态: 保护关闭」，需要重新启用。

### 4.2 启用保护器

```powershell
manage-bde -protectors -enable C:
manage-bde -protectors -enable D:
manage-bde -protectors -enable E:
```

### 4.3 数据盘自动解锁

```powershell
manage-bde -autounlock -enable D:
manage-bde -autounlock -enable E:
```

### 4.4 GUI 方式（备选）

```powershell
control /name Microsoft.BitLockerDriveEncryption
```

在 BitLocker 控制面板中对每个驱动器点击「启用 BitLocker」或「恢复保护」。

### 4.5 验证

```powershell
manage-bde -status
```

目标：所有盘「保护状态: 保护已启用 ✓」「锁定状态: 已解锁」「自动解锁: 已启用」

---

## 验证最终结果

```powershell
Get-Partition -DiskNumber 0 | Format-Table PartitionNumber, DriveLetter, @{N="SizeGB";E={[math]::Round($_.Size/1GB,1)}}, Type
Get-Volume -DriveLetter C, D, E | Format-Table DriveLetter, @{N="SizeGB";E={[math]::Round($_.Size/1GB,1)}}, @{N="FreeGB";E={[math]::Round($_.SizeRemaining/1GB,1)}}
manage-bde -status
```

---

## 常见问题

| 问题 | 解决方法 |
|------|----------|
| 忘了点「套用」就重启了？ | 操作没有生效，重启后分区不变，需重新操作 |
| AOMEI 中排错操作了？ | 点击「撤销」按钮或按 Ctrl+Z |
| 分区移动后无法启动？ | Windows 恢复环境 → `bootrec /fixmbr` + `bootrec /fixboot` |
| BitLocker 提示输入恢复密钥？ | 使用 48 位恢复密钥（在 Microsoft 账户中查找） |
| manage-bde 拒绝访问？ | 以管理员权限运行 PowerShell |
| 保护器显示「找不到」？ | 先添加：`manage-bde -protectors -add D: -recoverypassword` |

---

## 注意事项

1. **确保 F 盘内无数据再删除**，删除前通过 `Get-Volume` 或资源管理器反复确认目标分区为空或已备份，分区一旦删除数据不可恢复
2. **磁盘操作工具下载安装好后，要手动操作移动和扩大 C 盘**，不要让 AI 操作。AOMEI 的 MFC 界面 AI 无法可靠自动化，反复尝试只会消耗 Token 并陷入死循环
3. **BitLocker 在移动 E 盘前要解密打开**（`manage-bde -protectors -disable`），在扩大 C 盘点击「套用」重启电脑后，要重新开启 BitLocker 保护（`manage-bde -protectors -enable`）以保证安全
4. **操作前务必备份重要数据**
5. **笔记本必须接通电源**，断电可能导致数据丢失
6. **PreOS 执行期间不要强制关机**
7. **「套用」按钮必须点击**，否则所有拖拽操作不会生效
8. 本次只扩展 C 盘右侧边界，**不移动 C 盘起始位置**，相对安全
