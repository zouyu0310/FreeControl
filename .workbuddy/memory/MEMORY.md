# FreeControl 项目笔记

## 项目定位
基于 scrcpy 的 Windows 桌面 GUI 工具，用于在 PC 上投屏+控制 Android 设备（USB / 无线）。当前版本 v1.7.3，内置 scrcpy v3.0 (win32)。

## 技术栈
- 语言/框架：C# + Windows Forms（WinForms）
- UI 皮肤库：SunnyUI 3.0.3
- 目标框架：.NET Framework 4.7.2（开发工具 Visual Studio 2022）
- 打包：Costura.Fody 5.7.0（把依赖 DLL 嵌入单 exe）
- 单项目解决方案（FreeControl.csproj），嵌入依赖而非多项目

## 核心运行流程
入口 Program.Main → 单例主窗体 `Singleton<Main>.Instance`
→ 用户点「启动」(Main.StartButtonClick) 拼接启动参数 List<string> StartParameters
→ RunScrcpy() 用 Process.Start 启动内置 `scrcpy.exe` 并监听其输出/退出
→ 控制器 Controller 悬浮窗通过 ADB.ExecuteShell 发送 `input keyevent` 指令（KeyCode 枚举）

## 关键文件地图
- Program.cs：入口 + 版本号常量 (1.7.3)
- Main.cs（最大，~42KB）：主窗体、配置读写、scrcpy 启动/监听、无线连接、输入法切换、多语言
- Setting.cs：配置数据模型（所有可配置项，[Description] 对应 scrcpy 参数名）——加新设置的首要改点
- Controller.cs：悬浮虚拟按键窗（Home/Back/Menu/Screenshot/音量/静音/电源）
- Trusteeship.cs：托管/批量控制窗体
- Utils/ADB.cs：adb.exe 封装 + KeyCode 枚举
- Utils/MoveListener.cs：监听 scrcpy 窗口移动，让控制器吸附
- Utils/{Logger,LogHelper,MailHelper,FileHelper,JsonHelper,Extend(Singleton<T>),SysEnvironment(注册表PATH)}.cs
- Resources/scrcpy-win32-v3.0.zip：内置 scrcpy，首次运行解压到 %AppData%\FreeControl\
- FodyWeavers.xml + packages.config（NuGet 依赖清单）

## 配置与数据
- 配置文件：`%AppData%\FreeControl\config.json`（JsonHelper 用 System.Web.Script.Serialization）
- scrcpy 路径：%AppData%\FreeControl\scrcpy-win32-v3.0\ ；支持 CustomScrcpyPath 指向外部 scrcpy
- 截图保存：桌面\FreeControl Screenshots

## 二次开发切入点
- 升级 scrcpy：替换 Resources 的 zip + Main.cs 的 ScrcpyVersion 常量
- 新增设置项：Setting.cs 加属性（[Description] 参数名）→ Main.cs 启动参数拼接处加逻辑 → Main.Designer.cs 加控件
- 新增控制器按钮：Setting.ControllerButton 列表 + Controller.InitButton + KeyCode 枚举
- 多语言：Main.resx / Main.en.resx；语言枚举 Lang.zh_cn/en
- 构建参考：.github/workflows/build-and-release.yml（本地编译失败按此流程补依赖）
