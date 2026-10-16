# ZYM_personal_plug~MatrixSwitcher

通用矩阵切换器 Q-SYS 插件（qplug）。通过 TCP 对接外部矩阵/切换器，用 `{I}` / `{O}` 占位符模板拼接切换命令并发送。

- **版本**：1.0.10
- **作者**：Zhouyumou
- **平台**：Q-SYS Designer（Lua 5.3 脚本插件）
- **规模**：默认 8×8，输入/输出各 1–16 可配

## 功能

- 可配置输入/输出数量（1–16），面板与引脚按数量自动生成
- TCP 客户端连接矩阵设备，支持自动重连（看门狗 + `ReconnectTimeout`）
- “先选输出、再选输入”的路由交互（arm-then-route 状态机）：输出保持高亮，对应输入点亮
- 每通道可命名、指定切换代码、选图标，按钮上显示图标 + 名称标签
- Config 页图标下拉含 Q-SYS 全套图标（234 个），并提供实时图标预览
- 切换命令经引脚引出：原始文本（`Cmd Out Raw`）与 `\xNN` 转义（`Cmd Out Hex`，可直接接 Command Buttons）
- `Send Trig` 触发引脚：命令变化时脉冲，可接 Command Buttons 自动发送
- `Disable Internal Send` 属性：关闭插件内部 TCP 发送，避免与外部发送器重复发

## 安装

1. 把 `MatrixSwitcher.qplug` 复制到 Q-SYS Designer 的插件目录：
   `我的文档\QSC\Q-Sys Designer\Plugins\`
2. **彻底退出** Q-SYS Designer 再重开（Designer 按 `插件 Id + 版本` 缓存编译结果，只删实例重拖会沿用旧快照）。
3. 在 Design 里拖入插件实例，点 **Emulate** 运行。

## 配置（Properties）

| 属性 | 类型 | 默认 | 说明 |
|---|---|---|---|
| Num Inputs | integer 1–16 | 8 | 输入通道数 |
| Num Outputs | integer 1–16 | 8 | 输出通道数 |
| IP Address | string | 192.168.1.10 | 矩阵设备 IP |
| Port | integer 1–65535 | 23 | TCP 端口（telnet 类常用 23） |
| Auto Reconnect | boolean | true | 断线自动重连 |
| Reconnect Interval | integer 1–60 | 5 | 重连间隔（秒） |
| Disable Internal Send | boolean | false | true = 只走引出引脚、不发内部 TCP |

## 使用

### 路由（Main 页）

1. 点一个 **Output** 按钮选中输出（radio 式，再点一次取消）。
2. 点一个 **Input** 按钮，把该输入路由到当前选中的输出；旧输入灭、新输入亮，输出保持高亮。
3. 插件按 `Cmd Template` 拼出切换命令并发送（已连接且未禁用内部发送时）。

### 通道命名 / 代码 / 图标（Config 页）

- `Input/Output Name`：通道显示名（显示在按钮下方标签）。
- `Input/Output Code`：切换代码（替换模板里的 `{I}` / `{O}`）。
- `Input/Output Icon`：从 Q-SYS 全套图标里选；`Icon Prev` 列实时预览所选图标。
- `Cmd Template`：切换命令模板，支持占位符与转义（见下）。

### 命令模板语法

路由时模板中的占位符被替换：

- `{I}` → 当前输入通道的 `Input Code`
- `{O}` → 当前输出通道的 `Output Code`

模板内可直接写转义：

- `\r` 回车、`\n` 换行
- `\xNN` 两位十六进制 → 真实字节（如 `\x0D\x0A`）

**示例模板**：`SET OUT {O} IN {I}\r\n`
路由 In3 → Out2 且代码为空时，发送：`SET OUT 2 IN 3\r\n`

## 控制引脚（UserPin）

| 引脚 | 方向 | 类型 | 说明 |
|---|---|---|---|
| Input / Output | Both | Toggle Button（数组） | 路由按钮 |
| Connect | Both | Toggle | TCP 连接开关 |
| Status | Output | Toggle | 连接状态 |
| Status_Text | Output | Text | 状态文本 |
| IP_Display | Output | Text | 当前 IP（内部显示） |
| Armed | Output | Text | 当前选中输出 |
| Log | Output | Text | 运行日志 |
| Cmd Out Raw | Output | Text | 切换命令原始字节 |
| Cmd Out Hex | Output | Text | `\xNN` 转义命令（接 Command Buttons） |
| Send Trig | Both | Trigger | 命令变化脉冲 |
| Clear Log | Both | Trigger | 清空日志 |

## 注意事项

- Q-SYS Designer 按 `插件 Id + Version` 缓存编译结果。**每次修改 `.qplug` 后必须 bump `PluginInfo.Version` / `BuildVersion`** 并彻底退出 Designer 重拖，否则面板不更新。
- `TcpSocket.Reconnect` 是“正在重连”事件回调（function），不是布尔开关；自动重连由 `ReconnectTimeout`（秒，0 = 禁用）控制。
- 本插件是通用模板，命令格式需按你的矩阵设备协议调整 `Cmd Template`。

## 许可

本插件为作者原创的 Q-SYS Lua 代码，供学习与集成参考使用。Q-SYS / Q-SYS Designer 为 QSC 商标。
