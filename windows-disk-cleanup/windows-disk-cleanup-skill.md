---
name: windows-disk-cleanup
description: Windows 自带磁盘清理工具（cleanmgr.exe）的完整操作指南，覆盖 GUI 每项清理说明、管理员模式、命令行自动化、Dism 后台清理与多合一脚本，适合日常磁盘维护。
---

# 🧹 Windows 自带磁盘清理 —— 你可能一直在浪费空间

> 不用装任何第三方软件。Windows 自带的 **磁盘清理 (cleanmgr.exe)** 就是最安全、最高效、最被低估的系统清理工具。
>
> 本文逐项拆解每一个清理选项，告诉你哪些可以放心勾、哪些要留个心眼，末尾附闭环的 BAT/PowerShell 一键脚本。

![GitHub release](https://img.shields.io/badge/Windows-11-0078D4?style=flat-square&logo=windows11)
![tool](https://img.shields.io/badge/tool-cleanmgr.exe-orange?style=flat-square)
![license](https://img.shields.io/badge/license-MIT-green?style=flat-square)
![PRs](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat-square)

---

## 为什么你需要看这篇

Windows 内置的磁盘清理工具诞生于 Windows 98 时代，至今仍在 Windows 11 中服役。绝大多数用户要么不知道它的存在，要么只会点"确定"懒得细看。结果就是 ——

> **一个 C 盘常年红条，却浪费着 10~50 GB 的可清理空间。**

本文适合：
- 日常用户 —— 看懂每个清理项，放心勾选
- 帮亲友修电脑的人 —— 一键脚本甩过去
- 内容创作者 —— 可直接转载/改编发布

---

## ⚡ 30 秒速成

```
Win + R → 输入 cleanmgr → 回车
             ↓
      选 C 盘，确定
             ↓
   勾上所有想清理的项目
             ↓
  点击「清理系统文件」（释放大头空间的关键！）
             ↓
  再次勾选 → 确定 → 删除文件
```

> 🎯 **核心一步**：左下角那个「清理系统文件」按钮 —— 不点它，你最多只能清理零头。点了它，Windows 更新缓存、Windows.old、驱动备份这些 **10~30 GB 级别的大户**才会出现。

---

## 🖥️ 界面详细拆解

> ℹ️ 以下基于 **Windows 11 23H2** 中文版实际界面描述。Windows 10 用户界面基本一致。

### 📍 基础模式（不点「清理系统文件」）

打开 cleanmgr 后首先出现的是**标准模式**，每个清理项左侧有一个复选框，右侧显示该项预估可释放空间。底部有三个按钮：「清理系统文件」「查看文件」「确定」。

在此模式下你通常会看到：

| 界面中的名称 | 实际是什么 | 安全？ |
|---|---|---|
| **回收站** | `$Recycle.Bin` 中尚未永久删除的文件 | ✅ 完全安全 |
| **下载的程序文件** | IE 时代的 ActiveX / Java 小程序残留 | ✅ 完全安全（现代系统基本为空） |
| **Internet 临时文件** | IE / Edge 的网页缓存 | ✅ 安全 |
| **DirectX 着色器缓存** | 游戏/图形渲染缓存 | ✅ 安全（下次启动游戏会重建） |
| **传递优化文件** | Windows Update 的 P2P 分发缓存 | ✅ 安全（1~5 GB） |
| **缩略图** | 资源管理器缓存的图片缩略图数据库 (`thumbcache_*.db`) | ✅ 安全（自动重建） |
| **临时文件** | `%TEMP%` + `C:\Windows\Temp` 下的临时数据 | ✅ 安全 |
| **Windows 错误报告** | WER 归档的系统/应用崩溃报告 | ✅ 安全 |

### 📍 管理员模式（点击「清理系统文件」后）

这一步 **需要管理员权限**，点击按钮后会弹出 UAC 确认，然后重新扫描。新增的关键项目：

| 界面中的名称 | 实际是什么 | 安全？ | 典型容量 |
|---|---|---|---|
| **Windows 更新清理** | `SoftwareDistribution\Download` 中已安装更新的旧版本副本 | ✅ 安全 | **5~30 GB** |
| **Windows 升级日志文件** | `CBS.log` 等升级过程中产生的日志 | ✅ 安全 | 数百 MB ~ 数 GB |
| **以前的 Windows 安装** | `C:\Windows.old`（大版本升级后保留的回滚备份） | ⚠️ **确认新系统稳定后再清** | **10~30 GB** |
| **设备驱动程序包** | `DriverStore\FileRepository` 中旧版本驱动 | ⚠️ 保留最新版即可 | 数百 MB ~ 数 GB |
| **系统还原和卷影复制** | 除最近一次之外的所有系统还原点 | ⚠️ 保留最近一个还原点 | 视配置而定 |

> 💡 **空间释放排行榜**
>
> 🥇 以前的 Windows 安装 —— **10~30 GB**
> 🥈 Windows 更新清理 —— **5~30 GB**
> 🥉 临时文件 —— **1~10 GB**
> 4️⃣ 传递优化文件 —— **1~5 GB**
> 5️⃣ 系统还原点 —— **按配额，可能数 GB**

---

## 📦 每项深度说明

### 回收站

- **磁盘路径**：`C:\$Recycle.Bin\<用户 SID>\`
- **清理什么**：你按 Delete 删除但尚未 Shift+Delete 或清空回收站的文件
- **清理后效果**：文件永久删除，**不可恢复**
- **建议**：定期清。如果回收站常年不清理且积压了上 GB 的东西，勾上。

### 下载的程序文件

- **磁盘路径**：`C:\Windows\Downloaded Program Files`
- **清理什么**：Internet Explorer 时代的 ActiveX 控件和 Java Applet
- **谁还会用到**：几乎没人。现代浏览器（Edge/Chrome/Firefox）完全不依赖这个目录。
- **建议**：闭眼勾。

### Internet 临时文件

- **磁盘路径**：`C:\Users\<用户名>\AppData\Local\Microsoft\Windows\INetCache`
- **清理什么**：IE 和旧版 Edge 缓存的网页静态资源（CSS/JS/图片）
- **注意**：不会清理 Chrome/Firefox 的缓存，那些在各自 AppData 目录里。
- **建议**：安全清理。重度 Edge 用户可释放数百 MB。

### DirectX 着色器缓存

- **磁盘路径**：`C:\Users\<用户名>\AppData\Local\Microsoft\DirectX Shader Cache`
- **清理什么**：GPU 渲染编译后的着色器二进制文件
- **清理后效果**：下次启动游戏/3D 应用的加载时间会**略微延长**（重建缓存），运行时性能不受影响
- **建议**：空间充裕就留着（能加快游戏加载）；空间紧张就清。

### 传递优化文件

- **磁盘路径**：`C:\Windows\SoftwareDistribution\DeliveryOptimization`
- **清理什么**：Windows Update 的 P2P 分发机制缓存的更新包。你的电脑在下载更新的同时也会把更新块分享给局域网/互联网上其他电脑，这些块临时存于此处。
- **建议**：安全清理。不想让电脑当 P2P 节点可以在「设置 → Windows 更新 → 高级选项 → 传递优化」中关闭。

### 缩略图

- **磁盘路径**：`C:\Users\<用户名>\AppData\Local\Microsoft\Windows\Explorer\thumbcache_*.db`
- **清理什么**：文件资源管理器为图片/视频文件夹生成的缩略图缓存
- **清理后效果**：下次打开包含大量图片的文件夹时缩略图变慢（重新生成），但不影响文件本身
- **建议**：安全清理。缩略图缓存日积月累可膨胀到数 GB。

### 临时文件

- **磁盘路径**：
  - 系统级别：`C:\Windows\Temp`
  - 用户级别：`C:\Users\<用户名>\AppData\Local\Temp`
- **清理什么**：安装程序解压的临时包、应用运行中的中间文件、日志片段等
- **安全细节**：Windows 默认只清理 **7 天以上未修改**的临时文件，所以刚产生的不会被删。这个逻辑很保守。
- **建议**：最安全的清理项之一，放心勾。

### Windows 错误报告文件

- **磁盘路径**：
  - 系统归档：`C:\ProgramData\Microsoft\Windows\WER\ReportArchive`
  - 用户归档：`C:\Users\<用户名>\AppData\Local\Microsoft\Windows\WER\ReportArchive`
  - 排队中：`C:\ProgramData\Microsoft\Windows\WER\ReportQueue`
- **清理什么**：程序崩溃/系统蓝屏时 Windows Error Reporting 收集的诊断数据（.dmp 文件、日志）
- **谁需要保留**：软件开发者、IT 运维排查问题
- **建议**：普通用户直接清。对排查崩溃没有需求就不用留。

### Windows 更新清理 ⭐

- **磁盘路径**：`C:\Windows\WinSxS` 中的旧版本组件 + `C:\Windows\SoftwareDistribution\Download` 的已安装更新包
- **清理什么**：Windows 每安装一个累积更新，都会保留旧版系统文件以便回滚。这些备份日积月累是 C 盘空间的最大吞噬者。
- **清理后效果**：**无法卸载已安装的 Windows 更新**。如果系统稳定运行一段时间后，这项清理非常安全。
- **建议**：每月清理一次。这是日常维护中释放空间最大的单项操作。

### Windows 升级日志文件

- **磁盘路径**：`C:\Windows\Logs\`、`C:\Windows\Panther\` 等
- **清理什么**：Windows 大版本升级（如 22H2 → 23H2）过程中生成的大量日志
- **建议**：系统正常升级完成后即可清理。升级失败了才需要保留日志排查。

### 以前的 Windows 安装 ⚠️

- **磁盘路径**：`C:\Windows.old\`
- **清理什么**：从旧版 Windows 升级到新版时，安装程序会把整个旧系统复制到 Windows.old 作为回滚手段
- **清理后效果**：**无法通过「设置 → 恢复 → 回退」回到旧版 Windows**
- **时间限制**：Windows 会在 **10 天后自动删除** Windows.old。如果你想手动提前清理：
  1. 确认新系统运行稳定
  2. 所有软件/驱动正常工作
  3. 你确定不需要回退
- **建议**：升级后至少等一周，确认一切正常后再清。

### 设备驱动程序包 ⚠️

- **磁盘路径**：`C:\Windows\System32\DriverStore\FileRepository`
- **清理什么**：旧版本设备驱动的安装包备份
- **注意**：只会清理**当前未在使用**的旧版驱动。当你的外设（打印机、显卡、蓝牙设备）工作正常时，清理旧版没问题。
- **建议**：如果近期没有更换/添加新硬件，可以清理。如果某外设刚好出问题，保留驱动便于排查。

### 系统还原和卷影复制 ⚠️

- **这是什么**：系统还原点 + 卷影复制快照（文件历史/备份服务的一部分）
- **清理什么**：除**最近一次**还原点之外的所有历史还原点
- **清理后效果**：仅保留最新还原点，无法回退到更早的状态
- **建议**：如果 C 盘空间极度紧张，清理旧还原点能释放数 GB。但强烈建议保留至少一个还原点。

---

## 🤖 命令行：从手动到全自动

如果你帮亲友维护电脑，手动点 GUI 效率太低。下面涵盖从单个命令到多合一脚本的全部方案。

### 快速参考卡

```batch
REM ── 图形界面 ──────────────────────────
cleanmgr                        :: 打开 GUI（需选盘符）
cleanmgr /d C:                  :: 跳过选盘符，直接扫 C 盘

REM ── 静默运行（需要先配一次） ──────────
cleanmgr /sageset:1             :: 打开 GUI 勾选配置 → 保存为配置 1
cleanmgr /sagerun:1             :: 后台静默执行配置 1（无弹窗）

REM ── 按钮直达管理员模式 ────────────────
cleanmgr /d C:                  :: 打开后再点「清理系统文件」

REM ── Dism 后台清理（更彻底）────────────
Dism /online /Cleanup-Image /StartComponentCleanup
                                :: 清理 WinSxS 旧组件（常规）
Dism /online /Cleanup-Image /StartComponentCleanup /ResetBase
                                :: 深度清理（无法卸载已装更新）
Dism /online /Cleanup-Image /SPSuperseded
                                :: 清理 Service Pack 备份（Win10 早期）

REM ── 其他辅助命令 ─────────────────────
ipconfig /flushdns              :: 刷新 DNS 缓存
wsreset.exe                     :: 清理 Microsoft Store 缓存
sfc /scannow                    :: 检查系统文件完整性
chkdsk C: /f                    :: 检查并修复磁盘错误（需重启）
```

### sageset / sagerun 原理

`cleanmgr.exe` 支持 **0~65535** 共 65536 个独立配置档。每个配置档可以记录不同组合的勾选项：

```
配置 1 (sageset:1)：日常维护 → 勾 临时文件 + 回收站 + 缩略图
配置 2 (sageset:2)：深度清理 → 勾 全部（含 Windows.old + 更新缓存）
配置 3 (sageset:3)：仅用户文件 → 勾 回收站 + Internet 缓存
```

操作步骤：

1. **以管理员身份**打开命令提示符
2. 运行 `cleanmgr /sageset:1`
3. 在 GUI 中勾选想要的清理项 → 确定（此时不会实际清理，只是保存配置）
4. 以后只需运行 `cleanmgr /sagerun:1` 即可一键执行

> ⚠️ 配置保存在注册表 `HKLM\Software\Microsoft\Windows\CurrentVersion\Explorer\VolumeCaches\` 下，每台电脑独立。换电脑需要重新配置。

---

## 📜 一键脚本

以下脚本可保存为 `.bat` 文件，**右键 → 以管理员身份运行**。

### 🥇 标准版（推荐日常使用）

```batch
@echo off
title Windows 磁盘清理 - 一键维护
chcp 65001 >nul
color 0E

echo ╔══════════════════════════════════════╗
echo ║   Windows 自带磁盘清理 — 一键脚本  ║
echo ╚══════════════════════════════════════╝
echo.

:: ===== 第一步：磁盘清理 GUI 直通车 =====
echo [1/3] 正在打开磁盘清理（请手动点确定）...
echo       提示：记得点击左下角「清理系统文件」！
start /wait cleanmgr /d C:

:: ===== 第二步：Dism 组件清理 =====
echo.
echo [2/3] 正在后台清理 WinSxS 旧组件...
Dism /online /Cleanup-Image /StartComponentCleanup /Quiet
if %ERRORLEVEL% EQU 0 (
    echo       完成
) else (
    echo       请以管理员身份运行此脚本
)

:: ===== 第三步：辅助清理 =====
echo.
echo [3/3] 正在清理 DNS 缓存 & Store 缓存...
ipconfig /flushdns >nul
wsreset.exe >nul 2>&1

echo.
echo ══════════════════════════════════════
echo   清理完成。建议重启计算机。
echo ══════════════════════════════════════
pause
```

### 🥈 静默版（日常定时任务）

前提：已通过 `cleanmgr /sageset:1` 配置过。

```batch
@echo off
title 静默磁盘清理
:: 以管理员身份运行时才有效
cleanmgr /sagerun:1
Dism /online /Cleanup-Image /StartComponentCleanup /Quiet
exit
```

配合 Windows 任务计划程序，每月自动运行一次。

### 🥉 深度版（半年大扫除）

```batch
@echo off
title 磁盘深度清理
chcp 65001 >nul
color 0C

echo ╔══════════════════════════════════════╗
echo ║       磁盘深度清理（半年一次）      ║
echo ╚══════════════════════════════════════╝
echo.
echo 请注意：
echo  - 此脚本会清理 WinSxS 重置基础，之后无法卸载已装更新
echo  - 请确认系统稳定运行
echo.
pause

echo [1/4] 磁盘清理（系统文件）...
cleanmgr /sagerun:2

echo [2/4] WinSxS 深度清理...
Dism /online /Cleanup-Image /StartComponentCleanup /ResetBase

echo [3/4] 清理临时文件...
del /f /s /q "%WINDIR%\Temp\*" >nul 2>&1
del /f /s /q "%TEMP%\*" >nul 2>&1

echo [4/4] 清理回收站 & Prefetch...
rd /s /q C:\$Recycle.Bin >nul 2>&1
del /f /s /q C:\Windows\Prefetch\* >nul 2>&1

echo.
echo 深度清理完成。请重启计算机。
pause
```

### PowerShell 版本

```powershell
#Requires -RunAsAdministrator

Write-Host @"
╔══════════════════════════════════════╗
║   Windows 磁盘清理 — PowerShell     ║
╚══════════════════════════════════════╝
"@ -ForegroundColor Cyan

# 1. 磁盘清理（静默 — 需要已配置 sageset:1）
Write-Host "`n[1/4] 磁盘清理..." -ForegroundColor Yellow
try {
    Start-Process -FilePath "cleanmgr.exe" -ArgumentList "/sagerun:1" -NoNewWindow -Wait
} catch {
    Write-Host "  跳过（可能未配置 sageset:1）" -ForegroundColor DarkYellow
}

# 2. Dism 组件清理
Write-Host "[2/4] WinSxS 组件清理..." -ForegroundColor Yellow
Start-Process -FilePath "dism.exe" -ArgumentList "/online /Cleanup-Image /StartComponentCleanup /Quiet" -NoNewWindow -Wait

# 3. Temp 文件夹
Write-Host "[3/4] 临时文件..." -ForegroundColor Yellow
@("$env:WINDIR\Temp", "$env:TEMP", "$env:LOCALAPPDATA\Temp") | ForEach-Object {
    if (Test-Path $_) {
        Get-ChildItem -Path $_ -Recurse -Force -ErrorAction SilentlyContinue |
            Where-Object { -not $_.PSIsContainer -and $_.LastWriteTime -lt (Get-Date).AddDays(-7) } |
            Remove-Item -Force -ErrorAction SilentlyContinue
        Write-Host "  $_" -ForegroundColor Green
    }
}

# 4. 回收站
Write-Host "[4/4] 回收站..." -ForegroundColor Yellow
Clear-RecycleBin -Force -ErrorAction SilentlyContinue

Write-Host "`n全部完成！" -ForegroundColor Green
```

---

## 🏗️ 进阶：Dism 组件清理详解

很多人只知道 cleanmgr，不知道 Windows 还自带了一个更深层的清理引擎：**Dism（部署映像服务和管理工具）**。

### 命令层级

| 命令 | 做了什么 | 后果 |
|---|---|---|
| `Dism /online /Cleanup-Image /StartComponentCleanup` | 清理 WinSxS 中未被引用的旧组件版本 | 常规清理，不影响更新卸载 |
| `Dism /online /Cleanup-Image /StartComponentCleanup /ResetBase` | 将当前组件版本设为新的"基线" | **所有已装更新不可卸载** |
| `Dism /online /Cleanup-Image /SPSuperseded` | 删除 Service Pack 被取代的文件 | Windows 10 后期版本基本无此场景 |

### 什么时候用 /ResetBase？

```
你的 C 盘已经红条，且：
  ✅ 系统至少稳定运行了 2 周
  ✅ 所有更新已安装
  ✅ 你不打算卸载任何 Windows 更新
  ✅ 你做好了系统备份
     → 可以跑 /ResetBase
```

### 查看 WinSxS 实际大小

```batch
Dism /online /Cleanup-Image /AnalyzeComponentStore
```

输出示例：
```
Component Store (WinSxS) information:
Windows Explorer Reported Size : 12.34 GB
Actual Size                   : 11.87 GB
    Shared with Windows       :  8.21 GB
    Backups and Disabled      :  2.13 GB   ← 可清理部分
    Cache and Temporary       :  1.53 GB   ← 可清理部分
Date of Last Cleanup          : 2026-05-01
Number of Reclaimable Packages: 17
Component Store Cleanup Recommended : Yes   ← 提示你可以清理
```

---

## ⚖️ 为什么要用自带工具而不是 CCleaner？

| 维度 | 🏆 Windows 磁盘清理 | ⚠️ 第三方清理工具 |
|---|---|---|
| **安全性** | 微软维护，每个清理项都经过完整测试矩阵 | 历史上多次事故（误删注册表、捆绑恶意软件） |
| **系统文件清理** | WinSxS、更新缓存、Windows.old | 部分支持，通常不彻底 |
| **隐私** | 零遥测，不上传你的文件列表 | CCleaner 2017 年被植入后门，2021 年数据泄露 |
| **价格** | 免费，预装 | 基本版功能受限 / 专业版付费 |
| **注册表清理** | 不支持 —— 而且你也不需要 | 支持，但注册表清理带来的性能提升约等于零，风险却真实存在 |
| **浏览器清理** | 仅 IE/Edge | Chrome / Firefox / Brave 等全系支持 |
| **软件卸载** | 不支持（用控制面板） | 支持批量卸载 |

> 💡 **总结**：磁盘空间清理用 `cleanmgr` + `Dism`。浏览器缓存在各浏览器里手动清。软件卸载用系统自带的「应用和功能」。注册表不要去碰。免费、安全、不折腾。

---

## ❓ 常见问题

<details>
<summary><b>Q: 清理卡住了，一直转圈怎么办？</b></summary>

Windows 更新清理和 Windows.old 清理可能耗时 **10~30 分钟**，不是卡住，是真的在处理大量文件。

1. 打开任务管理器 → 找到 `cleanmgr.exe` → 看 CPU / 磁盘活动是否 > 0%
2. 有活动 → 等着，不要强制关机
3. 超过 1 小时且 CPU/磁盘均为 0% → 结束任务，重启后重试
</details>

<details>
<summary><b>Q: 清完了空间没怎么变，怎么回事？</b></summary>

三件套排查：
1. **忘了点「清理系统文件」**—— 大头都在管理员模式
2. **微信/QQ/企业微信**这些国产软件的聊天文件在 `Documents\WeChat Files` 和 `Documents\Tencent Files` 下，磁盘清理管不到
3. 用 **WizTree**（免费，2 秒扫完全盘）看是什么吃了空间

此外 `pagefile.sys`（虚拟内存）和 `hiberfil.sys`（休眠文件）也可能占 10~20 GB。用 `powercfg /h off` 关闭休眠可以干掉 hiberfil.sys。
</details>

<details>
<summary><b>Q: 「以前的 Windows 安装」没出现？</b></summary>

Windows 在升级后的第 10 天会自动删除 Windows.old。如果你升级超过 10 天了，这个选项自然消失。

如需手动提前删除：`cleanmgr /d C:` → 清理系统文件 → 勾选「以前的 Windows 安装」→ 确定。
</details>

<details>
<summary><b>Q: 清理后电脑变慢了？</b></summary>

正常现象，通常是这两个原因：
- **缩略图重建**：打开图片文件夹时稍慢，过一会就好了
- **Prefetch 重建**：如果你手动清了 Prefetch，前两次开机会慢，之后恢复
- 如果**持续变慢**：跟磁盘清理无关，检查 Windows Update 是否在后台装更新
</details>

<details>
<summary><b>Q: WinSxS 文件夹 20 GB，能直接删吗？</b></summary>

**绝对不要。** WinSxS 大部分是硬链接（hard link），并非真实占用两倍空间。直接删会导致系统不可启动。

正确做法：`Dism /online /Cleanup-Image /StartComponentCleanup`。
</details>

---

## 📅 维护日历

```
┌──────────────────────────────────────────┐
│              每周                         │
│  清空回收站（Del 键不放心就 Shift+Del）  │
└──────────────────────────────────────────┘
                    ↓
┌──────────────────────────────────────────┐
│              每月                         │
│  磁盘清理（标准模式）+ Temp 文件清理     │
└──────────────────────────────────────────┘
                    ↓
┌──────────────────────────────────────────┐
│              每季度                       │
│  磁盘清理（含系统文件）+ Dism 组件清理   │
└──────────────────────────────────────────┘
                    ↓
┌──────────────────────────────────────────┐
│         大版本升级后（立即）              │
│  确认稳定 → 清理 Windows.old             │
│  + Windows 更新缓存 + 升级日志            │
└──────────────────────────────────────────┘
```

---

## 🧰 配套工具箱

这些工具配合 Windows 磁盘清理使用，帮你搞清楚空间的去向：

| 工具 | 做什么 | 下载 |
|---|---|---|
| **WizTree** | 2 秒扫完全盘，按文件夹/文件大小可视化 | [diskanalyzer.com](https://diskanalyzer.com) |
| **Everything** | 即时搜索全盘文件，按大小排序找大文件 | [voidtools.com](https://www.voidtools.com) |
| **Dism++** | 第三方 Dism GUI，带系统优化与清理 | [GitHub](https://github.com/Chuyu-Team/Dism-Multi-language) |
| **WinDirStat** | 磁盘空间可视化（经典 treemap） | [windirstat.net](https://windirstat.net) |

---

## 📝 附录

### 清理项 → 磁盘路径速查

```
C:\$Recycle.Bin                             回收站
C:\Windows\Temp                             系统临时文件
C:\Users\<用户名>\AppData\Local\Temp        用户临时文件
C:\Windows\SoftwareDistribution\Download    Windows Update 下载缓存
C:\Windows\SoftwareDistribution\DeliveryOptimization   传递优化缓存
C:\Windows.old                              旧版 Windows 回滚备份
C:\Windows\System32\DriverStore\FileRepository         驱动备份
C:\Windows\Prefetch                          应用程序预加载缓存
C:\Windows\WinSxS                            组件存储（勿手动删！）
C:\ProgramData\Microsoft\Windows\WER\*       错误报告
C:\Windows\Logs                              系统日志
```

### cleanmgr 命令行参数完整列表

| 参数 | 行为 |
|---|---|
| `/d <盘符>:` | 跳过驱动器选择，直接分析指定盘 |
| `/sageset:<n>` | 打开 GUI 配置，保存为编号 n（0 ≤ n ≤ 65535） |
| `/sagerun:<n>` | 静默执行编号 n 的预设配置 |
| `/lowdisk` | 当磁盘空间低时自动运行（系统调用，一般不用手动触发） |
| `/verylowdisk` | 严重低空间时的自动清理模式 |
| `/setup` | 安全清理安装临时文件 |
| `/autoclean` | Windows 10 1809+ 新增，自动清理模式 |

> ⚠️ `/sageset` 和 `/sagerun` 后面的数字就是编号，只要两边的数字一致就行。推荐日常用 `1`，深度清理用 `2`。

---

## 📄 许可

MIT — 随意 fork、转载、改编。转载注明出处即可。

---

<p align="center">
  <b>维护系统整洁，从每次磁盘清理开始。</b><br>
  ⭐ 如果这篇指南对你有帮助，给个 Star 吧。
</p>
