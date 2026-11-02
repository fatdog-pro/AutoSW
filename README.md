[README.md](https://github.com/user-attachments/files/34254781/README.md)
# AutoSW 中文说明

作者：抖音 萌猪过河。

AutoSW 让具备本地命令或 MCP 能力的 Windows 智能体，通过 SOLIDWORKS 官方 API 完成参数化零件、装配、工程图及已有模型副本修改。模型理解自然语言和上传资料，AutoSW 负责受控执行、保存与原生读回；全程不使用屏幕识别、OCR、桌面点选、鼠标或键盘操控。

## 适用范围

- 从自然语言整理尺寸、基准、特征顺序与装配关系，生成可编辑的原生模型。
- 读取用户上传的图纸、图片和已有模型，在专用项目目录中建立或修改副本，保留原文件及引用。
- 查询本机实际安装的官方 API，按任务编写建模逻辑；简单示例不是通用脚本能力的上限。
- 保持接入时的同一个 SOLIDWORKS 进程与主窗口，后续要求复用连接，不因一次命令结束而关闭或新开软件。
- 用户没有要求单独展示零件时，用 `ctx.NewPart(name, false)` 创建后台零件，在主装配内通过原生在位编辑构造几何，只让主装配出现在前台；不要逐件 `ctx.Activate(part)`，也不要直接假定隐藏零件的几何调用可用。具体流程与已测边界见 [脚本上下文](references/native-api.md)。及时正常释放本轮已保存且暂不需要的独立零件文档。装配所需的隐藏引用仍占用必要内存，不把一个可见装配窗口解释为总文档数只能为 1。

已实测项目、未验证能力及精度边界见 [验证范围](references/verification.md)。接口存在不代表所有版本、任意复杂模型或 Simulation 求解均已通过；豆包客户端的实际本机接入能力也需要单独确认。

## 运行条件

使用 Windows x64、64 位 Python 3.10+、Windows PowerShell 5.1，以及本机已安装并激活的 SOLIDWORKS。所需模板、插件和许可由用户提供。技能包不包含 SOLIDWORKS、Python、厂商 DLL、SDK、序列号或测试模型；Python 入口仅使用标准库。

完整保留技能目录，不要只复制 `SKILL.md`。另建一个专用项目文件夹，将明确提供的图纸、图片和已有模型及其引用放在 `inputs/` 内。不要把整个磁盘、用户资料目录或技能目录设为项目工作区。

## 本地命令接入

以下 `<Python完整路径>`、`<技能目录>` 和 `<项目目录>` 都是占位符，必须替换为本机真实绝对路径；项目目录需预先存在。

```text
<Python完整路径> -I -B <技能目录>/scripts/autosw.py --workspace <项目目录> verify
<Python完整路径> -I -B <技能目录>/scripts/autosw.py --workspace <项目目录> init
<Python完整路径> -I -B <技能目录>/scripts/autosw.py --workspace <项目目录> connect
<Python完整路径> -I -B <技能目录>/scripts/autosw.py --workspace <项目目录> status
```

`verify` 检查桥接源码，`init` 准备项目会话；只有明确调用 `connect` 才能启动或附接 SOLIDWORKS。接入成功后继续复用原会话。具体命令、错误处理和恢复规则见 [命令与接口](references/cli.md)。

建模时先查询相关 API，使用 `write-script` 提交 UTF-8 C# `Build` 方法体，再以返回的精确 SHA 和稳定意图键调用 `run-script`。通过 `readback` 核对原运行编号的实际结果，通过 `open` 打开该运行已登记的成果。源码格式与可用上下文见 [原生 API 脚本](references/native-api.md)，完整步骤见 [建模与验证工作流](references/workflow.md)。

若原请求仍在运行或结果未知，只核对原编号，不反复建模、不换意图键重试、不另开软件。`status`、退出码 0 或宿主空闲本身都不能证明模型已经通过验收。

## MCP 接入豆包等智能体

当前宿主必须支持本机命令或本地 stdio MCP；普通网页聊天无法仅靠上传本技能控制本机软件。在支持本地 MCP 的客户端配置：

```json
{
  "command": "<本机64位Python完整路径>",
  "args": [
    "-I", "-B", "<技能绝对目录>/scripts/autosw.py",
    "--workspace", "<既存项目绝对目录>", "mcp"
  ]
}
```

服务应运行在安装 SOLIDWORKS 的同一 Windows 用户和登录会话内，无需开放公网端口。先确认客户端实际能列出并调用 `autosw_*` 工具，再显式接入、查询 API、提交源码及读回成果。协议和工具参数见 [命令与接口](references/cli.md)，客户端限制见 [豆包及其他本地智能体接入](references/doubao.md)。本说明不代表已在豆包的某个具体版本实测通过。

## 示例与交付

| 示例 | 内容 | 状态与依据 |
| --- | --- | --- |
| [原生装配示例](references/example.md) | 板、销、真实配合、工程图与 STEP | 实测范围见示例及验证文档 |
| [概念火箭示例](references/rocket-example.md) | 15 种零件、45 个组件、44 个原生锁定配合；分级箭体、整流罩、尾翼、喷管与细节 | 2026-11-01 原生验收通过：后台零件、主装配逐步显示、保存卸载后磁盘重开，见 [验收摘要](references/rocket-baseline.json) |

火箭的完整 [复现源码](examples/rocket.cs) 随技能提供，是本次真实执行的 `Build` 方法体。它在可见主装配中逐特征、逐部件原生绘制，15 个零件文档保持隐藏；最终原生外包络为 1.80×6.62×1.80 m，主体直径 0.90 m。按 [火箭说明](references/rocket-example.md) 在独立项目目录提交源码，即可复用该流程；原生模型和预览在实际执行后生成，不随技能包预置。

交付以真实生成的项目输出文件夹为入口，保留原生 `SLDPRT`、`SLDASM`、实际工程图与导出文件。检查真实特征、实体、尺寸、组件和配合，分别说明当前内存读回与保存后磁盘重开是否通过。外观预览可辅助核对模型，但不能代替原生数据验收。

普通用户任务保留成果和原输入。`cleanup` 只正常关闭原运行中归属明确且已保存的文档，不删除文件；`disconnect` 只退役空闲本地宿主，默认保留 SOLIDWORKS。不得退出用户软件、关闭全部文档或强杀同名进程。

## 包内文件

- `SKILL.md`：智能体入口和工作原则。
- `scripts/`、`bridge/`：本地 CLI/MCP 与官方 API 执行桥。
- `references/`：命令、建模工作流、脚本上下文、接入方式及验证边界。
- `examples/`：可复用的 C# `Build` 方法体；使用前阅读对应示例说明。
- `tests/`：本地适配器测试，不代替本机 SOLIDWORKS 的真实模型验收。
