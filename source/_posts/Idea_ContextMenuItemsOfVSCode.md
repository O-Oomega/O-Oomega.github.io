---
title: 自动恢复右键VSCode打开文件夹
date: 2023-12-26 22:00:00
categories:
- Idea
tags:
- 突发奇想
---

因为用的VSCode“绿色版”，所以没有添加相关的注册表项，右键文件夹也没有“VSCode打开文件夹”的选项，所以每次看项目都很麻烦。

我写了个BAT文件，用于自动添加“VSCode打开文件夹”的选项。

```
@echo off
echo 警告: 错误的注册表修改可能会导致严重的系统问题，请您接下来一定输入正确的数据格式

choice /C YN /M "确定要继续吗？(Y/N): "
if errorlevel 2 goto end
if errorlevel 1 goto continue

:continue
echo 正在“添加右击文件夹使用vscode打开”...

@REM 获取VSCode的路径
set /p VSCodePath="请输入VSCode.exe的完整路径（例如 C:\Program Files\Microsoft VS Code\Code.exe）: "

@REM 检查 shell 项是否存在
reg query "HKEY_CLASSES_ROOT\Directory\shell" >nul 2>&1

@REM 如果 shell 项不存在，则创建
if %ERRORLEVEL% EQU 1 (
    echo 创建 shell 项...
    reg add "HKEY_CLASSES_ROOT\Directory\shell" /f
)

@REM 检查 VSCode 项是否存在
reg query "HKEY_CLASSES_ROOT\Directory\shell\VSCode" >nul 2>&1

@REM 如果 VSCode 项不存在，则创建
if %ERRORLEVEL% EQU 1 (
    @REM 创建 VSCode 项
    echo 创建 VSCode 项...
    reg add "HKEY_CLASSES_ROOT\Directory\shell\VSCode" /f
)

reg add "HKEY_CLASSES_ROOT\Directory\shell\VSCode" /ve /t REG_SZ /d "使用VSCode打开文件夹" /f

@REM 设置VSCode图标
echo 设置VSCode图标...
reg add "HKEY_CLASSES_ROOT\Directory\shell\VSCode" /v "Icon" /t REG_EXPAND_SZ /d "%VSCodePath%" /f

@REM 设置打开VSCode的命令
echo 设置打开VSCode的命令...
reg add "HKEY_CLASSES_ROOT\Directory\shell\VSCode\command" /f
reg add "HKEY_CLASSES_ROOT\Directory\shell\VSCode\command" /ve /t REG_SZ /d "%VSCodePath% \"%%1\"" /f

echo 添加完成。
pause
goto eof

:end
echo 操作已取消。
pause
```
<br />
在桌面新建一个TXT文件，将代码复制进去，再另存为到桌面，并且更改后缀为bat，将编码格式改为ANSI后，运行即可。

![Alt text](/images/Idea/Idea_ContextMenuItemsOfVSCode.png)
