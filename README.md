<div align="center">

# MFH6A

**基于 MaaFramework 的《极限竞速：地平线 6》自动化工具**

![license](https://img.shields.io/badge/license-AGPL--3.0-blue.svg)
![platform](https://img.shields.io/badge/platform-Windows%20x64-lightgrey.svg)
![MaaFramework](https://img.shields.io/badge/MaaFramework-5.16.1-informational.svg)
![UI](https://img.shields.io/badge/UI-MXU-8a2be2.svg)

</div>

## 这是什么

MFH6A 是一个 MaaFramework 项目：由 MaaFramework 识别游戏画面、按 pipeline 操作窗口，MXU 负责图形界面。

它不读内存、不改存档、不注入游戏进程——所有动作都是「看一眼画面、点一下按钮」，跟人坐在那儿玩没有区别。这也是 MaaFramework 系工具的通用做法：识别层是图像与 OCR，执行层是模拟输入。

> 当前处于骨架阶段。`Tutorial` 是跑通链路用的示范任务，真正的玩法流程还没写。别急着把它当成成品。

## 功能

| 状态   | 功能               | 说明                                                    |
| ------ | ------------------ | ------------------------------------------------------- |
| 已实现 | 窗口连接与任务框架 | 通过 MXU 选择游戏窗口、启动任务、查看实时截图与运行日志 |
| 演示   | `Tutorial`         | 一个最小的识别 + 完成流程，用来验证 pipeline 能否正常跑 |
| 规划   | AFK 跑圈           | 自动驾驶跑圈，识别结算界面后自动重开                    |
| 规划   | 赛季进度巡检       | 遍历季节赛列表，输出还差哪几周                          |

## 使用

### 直接运行

发布包解压后目录长这样：

```
MFH6A/
├── mxu.exe          MXU 图形界面
├── maafw/           MaaFramework 运行时（DLL）
├── interface.json   项目描述（MXU 读取）
└── resource/        资源包（pipeline、图片、OCR 模型）
```

双击 `mxu.exe` 启动，在右侧「选择窗口」里选中游戏客户端的窗口，然后勾任务、点「开始任务」。

**工作目录必须是解压根目录**：MXU 靠同级目录里的 `interface.json` 和 `maafw/` 找东西，换个地方启动会白屏或者连不上。

### 前置条件

- Windows 10/11 x64
- 游戏窗口不要被最小化。MaaFramework 抓的是窗口画面，缩到任务栏就抓不到
- 部分操作需要游戏窗口在前台，后台点击对独占全屏的游戏无效。游戏里把显示模式改成「窗口化」或「无边框窗口」更稳

## 常见问题

**选不到窗口？**
先确认游戏已经启动并且窗口没最小化，再点「选择窗口」右边的刷新按钮。MXU 列出的是当前所有可见窗口的标题。

**点了开始任务没反应？**
看右侧「运行日志」。pipeline 找不到要识别的画面时会一直重试并打 `recognition failed`，日志里能看出它卡在哪一步。

**识别总是不准？**
分辨率或缩放比例（DPI）变了会影响模板匹配。把 Windows 显示缩放设成 100% 或者重新截取模板图。

**能不能后台挂机？**
可以，但依赖窗口不最小化。真·后台（锁屏、切到别的用户）会因为抓不到画面而失效。

## 开发

本项目由 [create-maa-project](https://github.com/Windsland52/create-maa-project) 3.8.0 生成，使用 **agent** 模板（Python 自定义逻辑）+ **MXU** 运行时。

### 目录结构

```
MFH6A/
├── agent/                  Python Agent
│   ├── main.py             入口（含 Python 版本检查）
│   ├── agent_runtime.py    加载并注册 custom/ 下的模块
│   ├── custom/action/      自定义动作
│   ├── custom/reco/        自定义识别
│   ├── custom/sink/        自定义回调
│   └── utils/              PI 环境变量、参数解析、日志、路径工具
├── resource/base/
│   ├── pipeline/           pipeline 定义（JSON）
│   ├── image/              模板图
│   └── model/ocr/          OCR 模型
├── tasks/                  interface.json 引用的任务定义
├── interface.json          PI v2 项目描述
├── maa-project.json        create-maa-project 项目配置
├── tools/schema/           JSON Schema（供编辑器补全与校验）
└── runtimes/, plugins/, libs/   运行时资产，由 --update 生成，不入库
```

### 环境准备

```bash
uv sync                                   # Python 依赖
pnpm install                              # 开发工具（格式化、Schema 校验）
create-maa-project --update ocr-models    # OCR 模型（不入库）
create-maa-project --update maafw         # MaaFramework 运行时（不入库）
```

在 VS Code 里打开项目，`.vscode/tasks.json` 会自动跑一次 `uv sync` 和 `pnpm install --frozen-lockfile`。

### 本地调试

```bash
uv run python agent/main.py   # 以 AgentServer 形式启动，供 MXU 或 MaaSupport 插件调用
```

VS Code 里用 `.vscode/launch.json` 的 `Maa Agent: Debug` 配置，映射关系在 `maatools.config.mts`。

修改 pipeline 后不必重启 MXU，在界面里重新点任务即可；改的是 Python 代码就得重启。

### 添加 Python 依赖

```bash
uv add <包名>
create-maa-project --update python-deps
```

`uv add` 只改 `pyproject.toml`、`uv.lock` 和本地 `.venv`；`requirements.in` / `requirements.txt` 是由 `--update python-deps` 从 `pyproject.toml` 重新生成的。发布包按 `requirements.txt` 预装依赖，所以加完依赖必须跑这一步，并把 `pyproject.toml`、`uv.lock`、`requirements.in`、`requirements.txt` 一起提交——不然本地能跑、发布出来缺依赖。

### 发布

项目当前没启用 CI。要开的话：

```bash
create-maa-project --add github
```

之后推送 `v0.1.0` 这样的 tag 会触发 `.github/workflows/release.yml`，CI 里会重新拉 MaaFramework、MXU 和 OCR 模型再打包。

打包时版本号取自这个 tag，并写回包内 `interface.json` 的 `version`；仓库里那份 `interface.json` 保持 `v0.0.0-dev` 这个内部测试版本，不需要随每次发布手改。

## 相关项目

- [MaaFramework](https://github.com/MaaXYZ/MaaFramework) —— 本项目的识别与执行引擎
- [MXU](https://github.com/MaaXYZ/MXU) —— MaaFramework Next UI，本项目使用的图形界面
- [create-maa-project](https://github.com/Windsland52/create-maa-project) —— 项目脚手架，本项目由它生成
- [MaaCommonAssets](https://github.com/MaaXYZ/MaaCommonAssets) —— OCR 模型来源

## 免责声明

本项目是非官方的第三方工具，与 Microsoft、Playground Games、Turn 10 以及 Steam 均无关系。

它只做画面识别与模拟输入，不修改游戏文件、不读取内存、不改存档。但这不意味着使用它没有风险：游戏运营方对自动化的态度、以及相关服务条款的判定，随时可能变化，账号处置的后果由使用者自行承担。

请把它当成学习 MaaFramework 的一个练手项目，而不是「稳定刷资源的外挂」。游戏是用来玩的，别让工具替你玩。

## License

[AGPL-3.0](./LICENSE)

---

English documentation: [README.en.md](./README.en.md)
