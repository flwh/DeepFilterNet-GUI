# DeepFilterNet3 GUI (WPF-UI)

基于 WPF-UI 的实时语音降噪桌面客户端，使用仓库内置的 Rust bridge 和 DeepFilterNet runtime 进行推理。程序自带默认模型，不再依赖外部 `df.dll` 或 `Models\DeepFilterNet3_onnx.tar.gz`。

**界面预览**
![主界面](docs/images/deepfilternet3-main-window.png)

**致谢**
- DeepFilterNet：`https://github.com/Rikorose/DeepFilterNet`

**亮点**
- 实时降噪：麦克风输入 -> 内置 DeepFilterNet runtime -> 扬声器输出。
- 内嵌默认模型：启动无需额外放置模型包。
- 多通道处理：输入支持双声道时默认按双声道处理，不再先混成单声道。
- 内部重采样：GUI 仅在输入侧对齐到输出采样率，runtime 内部自动完成与 48 kHz 模型采样率之间的转换。
- 后端与设备可选：WDM / MME / KS。
- 监控与可视化：波形、RTF、帧耗时、推理耗时、处理通道模式。
- 托盘常驻、开机启动、文件日志。

**环境要求**
- Windows 10/11 x64
- .NET 10 SDK
- 可用音频驱动（WDM/MME/KS）

桥接 DLL（`deepfilter_runtime_bridge.dll`，内嵌默认模型，约 24 MB）已预编译并随仓库放在 `prebuilt\`，日常构建不需要 Rust 工具链，也不需要初始化 Git 子模块。

**构建与运行**
```bash
dotnet restore
dotnet build
dotnet run --project DeepFilterNetGui/DeepFilterNetGui.csproj
```

`dotnet build` / `dotnet publish` 会直接把 `prebuilt\deepfilter_runtime_bridge.dll` 复制到输出目录，跳过 `cargo build`。

**重新编译桥接（仅改动 Rust 时才需要）**

日常构建不再需要 Rust 源码，DeepFilterNet 子模块也已从仓库移除。要重新编译时：

1. 安装 Rust 工具链（`x86_64-pc-windows-msvc`），并手动取回源码：
   `git submodule add https://github.com/Rikorose/DeepFilterNet.git extern/DeepFilterNet`
2. 删除或改名 `prebuilt\` 目录，构建会自动回退到 `cargo build`，产物取自 `rust\deepfilter_runtime_bridge\target\...`。
3. 编译完成后把新的 `deepfilter_runtime_bridge.dll` 放回 `prebuilt\`，恢复快速构建。

**依赖包**

| 包 | 版本 |
| --- | --- |
| `WPF-UI` | 4.2.0 |
| `WPF-UI.Tray` | 4.2.0 |
| `NAudio.Wasapi` | 2.2.1 |
| `NAudio.WinMM` | 2.2.1 |
| `PortAudioSharp2` | 1.0.6 |

**推理参数说明**

| 项目 | 说明 |
| --- | --- |
| 模型 | 使用程序内嵌的默认 DeepFilterNet3 模型 |
| 模型采样率 | 48 kHz |
| 帧长 | 由内置 runtime 返回，当前为 480 |
| 通道模式 | `Mono` / `Stereo`，默认优先双声道 |
| `ReduceMask` | `Independent (NONE)` / `Maximum (MAX)` / `Mean (MEAN)` |

**使用流程**

1. 选择输入/输出后端与设备。
2. 点击开始按钮启动实时推理。
3. 运行时可调降噪强度、后滤波强度；`ReduceMask` 在设置页调整并持久化。
4. 状态区会显示当前处理通道模式、采样率和性能指标。

**音频后端说明**

| 后端 | 说明 |
| --- | --- |
| WDM | WASAPI 共享模式，兼容性较好。 |
| MME | 传统接口，延迟较高。 |
| KS | 通过 PortAudio WDM-KS，依赖驱动支持。 |

**配置文件**

配置文件路径：`deepfilternet3.settings.json`（与 exe 同目录）。

| 字段 | 说明 |
| --- | --- |
| `EnableFileLogging` | 是否输出日志文件 |
| `EnableAutoStart` | 是否启用开机启动 |
| `MinimizeToTray` | 最小化到托盘 |
| `CloseToTray` | 关闭到托盘 |
| `StartToTray` | 启动到托盘 |
| `LastInputBackend` | 上次输入后端 |
| `LastOutputBackend` | 上次输出后端 |
| `LastInputDeviceId` | 上次输入设备 |
| `LastOutputDeviceId` | 上次输出设备 |
| `AudioSampleRate` | 优选采样率 |
| `DenoiseAttenLimitDb` | 降噪强度 |
| `PostFilterBeta` | 后滤波强度 |
| `ReduceMask` | 通道掩码合并策略 |

**日志文件**

文件日志默认关闭，开启后输出到 `logs/deepfilternet3-*.log`。

**常见问题**

1. 构建时报找不到 `cargo` 或 DeepFilterNet 源码
   日常构建不需要 Rust。只有在 `prebuilt\deepfilter_runtime_bridge.dll` 缺失时才会回退到 `cargo build`，按上文"重新编译桥接"操作或恢复该文件即可。
2. KS 启动失败
   驱动可能不支持当前格式或独占模式，建议切换 WDM/MME 进行验证。
3. 没有声音或输出很小
   检查设备音量、输入输出设备选择和驱动采样率支持情况。

**已知限制**

- 仅支持 CPU 推理。
- 当前 GUI 仅暴露单幅波形监视图；双声道模式下监视数据为双声道平均结果。
- 运行中不支持切换模型，仅使用内嵌默认模型。

**许可**

本项目采用双许可证（MIT 或 Apache-2.0），详见根目录 `LICENSE` / `LICENSE-MIT` / `LICENSE-APACHE`。DeepFilterNet 上游子模块继续遵循其各自许可证。
