# 图灵AI辅助

> ## ⚠️ 重要声明（请务必阅读）
>
> **本项目仅限在本地环境(localhost)中运行，禁止用于任何违反服务条款或法律法规的场景。**
>
> 本仓库**仅供安全研究、学习和本地测试的开发者工具**使用，仅供学习交流，非商业产品，非作弊工具宣传。使用者必须是具备相应技术的开发人员，并承诺仅在本机环境进行学习与研究。
>
> 任何在未经授权场景下的使用行为，与本项目作者和维护者无关，一切后果由使用者自行承担。

「图灵AI辅助」是一款注入浏览器的 AI 辅助研究引擎 —— 自动监听网页图灵测试的全流程，调用大模型 API 生成真人级回复并自动发送，实时分析对手是真人还是 AI，智能判定一键结算，并自动开启下一局。同时提供「任意门 · 自建网页客户端」配套发行：网页版被关闭也能在浏览器以微信身份完整游玩。

- 官网：https://klp-kulipa.github.io/turing-ai-toolkit/
- 当前版本：**V1.5**（免费、无内置广告）
- 检查更新：油猴脚本启动时会读取本站徽章上的版本号 `V1.5` 自动比对
- 客户端版本：**微信客户端 V1.0**

---

## 📋 目录

- [声明与定位](#-重要声明请务必阅读)
- [产品介绍](#-产品介绍图片展示)
- [工作原理](#-工作原理)
- [客户端支持](#-客户端支持微信)
- [支持的网站](#-支持的网站)
- [快速开始（主脚本）](#-快速开始主脚本)
- [快速开始（微信客户端）](#-快速开始微信客户端)
- [凭据续期](#-凭据续期jwt-约-7-天过期)
- [连线确认（防误安装）](#-连线确认防误安装)
- [免责声明](#-免责声明)
- [项目结构与发行包](#-项目结构与发行包)
- [官网源代码](#-官网源代码github-pages)

---

## 产品介绍（图片展示）

<img width="897" height="579" alt="image" src="https://github.com/user-attachments/assets/502504b7-41ff-46f9-a2ed-69b5a05a406f" />

### 核心能力

| 能力 | 说明 |
| --- | --- |
| 🧠 AI 辅助回复 | 接入任意 OpenAI 兼容大模型 API（内置 Agnes 纯公益站预设），实时生成真人级心理回复，支持连续对话链 |
| 🎯 真人 / AI 判定 | 实时分析对手行为特征，自动点击 H（真人）/ A（AI）判定按钮与确认弹窗，倒计时结束自动兜底结算 |
| ⚡ 主动跟进回复 | 对局静默超过 25 秒自动补一条消息防冷场，最多 2 次 / 局 |
| ♾️ 全自动连战 | 胜负平局自动识别，结算后自动回主页并自动开始下一局，无限连战 |
| 📊 战绩统计 | 胜负自动落库，可随时查看、导出、重置历史战绩 |
| 🔧 多维参数配置 | 输出温度 / 思考链 / 联网搜索 / 响应延迟动态模拟 / 广告植入 / JavaScript 脚本拓展，全套参数可导入导出 |
| 🧷 冷静的响应节奏 | 毫秒级延迟动态模拟（所有模式、所有连发空隙统一生效），配合防检测节奏 |
| 🖥️ 自建网页客户端 | 官方微信通道凭据注入，浏览器完整游玩（匹配 / 聊天 / 判定 / 返回房间），配套一键抓包工具 |
| 🛡️ 全程免费 | 脚本完全免费，无隐藏收费、无广告干扰（广告由你自己决定开关） |

### 工作流程

进入房间（自动识别站点状态并定位输入框与对话流）→ 监听对话（捕捉对方消息、判定按钮与倒计时）→ 生成回复（调用大模型生成伪装回复并模拟人工输入节奏）→ 判定结算（识别 H/A 按钮、确认弹窗、锁局，自动判定）→ 战绩与连战（统计胜负，自动开始下一局）。

## ⚙️ 工作原理

> 本节只解释代码逻辑与技术实现，供学习与研究参考，不构成任何使用指引。

### 1. 主脚本（油猴用户脚本）

- **注入与守卫**：脚本通过 `@match` 白名单仅匹配指定站点与 `localhost:8890`，其余页面不注入（`连线确认` 一节有完整清单）；本地端口非 8890 直接退出。
- **页面监听**：使用 `MutationObserver` 监听 DOM 变化，识别输入框、消息列表、判定按钮与倒计时元素（`SITE_ADAPTERS` 按站点注册选择器，未命中走通用 CSS / 文本启发式兜底）。
- **模型调用通道**：封装 `gmFetch` 双通道 —— 优先 `GM_xmlhttpRequest`（油猴特权请求，绕过浏览器同源限制），失败自动回退 `nativeFetch`，内置超时与 SSE / JSON 双格式解析。
- **回复发送**：将模型输出按配置的延迟范围（默认 500ms~3s 随机）分段模拟人工输入节奏写入输入框并触发发送事件。
- **状态机**：对局状态流转（排队 / 匹配 / 房间 / 结算）由消息订阅驱动，配合倒计时兜底与限流重试逻辑。
- **数据持久化**：配置与战绩存于浏览器 `GM_getValue / GM_setValue` 与 localStorage，无任何云端存储。

### 2. 自建网页客户端（本地代理架构）

- **本地代理**：`server.mjs` 基于 Node 内置模块（零 npm 依赖）实现 —— 静态页面服务 + HTTP API 转发 + WebSocket 代理。
- **WebSocket 代理原因**：官方服务器会掐断带 `Origin` 头的 WSS 连接（浏览器强制携带 Origin，实测 0 秒 code=1006 被断）；本地代理用 Node 无 Origin 连接上游，浏览器只连本地 `ws://localhost:8890/ws`，从而在本地环境完成协议联通。
- **凭据注入**：浏览器请求本地代理，代理在转发时附加三件套凭据（`Authorization` JWT / `X-Visitor-Id` / `X-Turing-Channel`），凭据保存在本地 `config.json`。
- **抓包工具**：`capture_tool.py` 在 `127.0.0.1:8888` 起 MITM 代理，仅解密 `anyanygame.com` 的微信流量以提取凭据，亦支持离线 HAR 导入兜底。

### 3. 联网搜索（可选后端）

- OpenSERP 本地服务监听 `127.0.0.1:7070`，脚本仅对本地地址发起搜索请求（百度引擎免费无需 Key）；本地服务未启动时功能自动失效，不产生任何外部请求。

> **网络行为说明**：所有大模型 API 请求均由使用者在控制面板主动操作触发（开启「自动回复」或手动发送）；脚本对 `localhost` 之外仅发起一次静态版本检查请求（见下节）。脚本本身不包含任何后台轮询、上报或遥测逻辑。

### 4. 版本检查机制

油猴脚本启动后（延迟 1.5s）会向本站发起一次 `GET` 请求读取版本徽章，与本地 `SCRIPT_VERSION` 比对后提示更新。该请求仅访问本仓库 GitHub Pages 静态页面，无其他网络交互。

## 🖥️ 客户端支持（微信）

| 客户端 | 状态 | 版本 | 说明 |
| --- | --- | --- | --- |
| 💬 微信客户端 | ✅ 已支持 | **V1.0** | 任意门「微信通道」游玩（控制中心 exe 内置抓包工具 + 服务器，零依赖开箱即用） |

> 微信客户端 = 控制中心 exe（`AnyAnygame.Wechat.Client.zip`）：内置 Node.js / openssl / 抓包工具，目标机器无需安装任何运行时；凭据自动捕获、到期提醒、服务器托盘常驻。

## 📦 包内容（微信客户端）

| 文件 | 说明 |
| --- | --- |
| `任意门图灵测试客户端.exe` | 控制中心：抓包工具 + 服务器两个入口（后台运行、系统托盘常驻） |
| `config.example.json` | 出厂配置模板（首次抓包后自动生成真实 `config.json`） |
| `README.md` | 说明文档（即本文档） |
| `src/` | 全部源码（供开发者 / 自查 / 命令行使用） |
| `src/抓包工具/` | 抓包工具源码与 openssl（exe 已内置） |

> ⚠️ **隐私说明**：出厂包不含任何账号数据。微信凭据（JWT）仅在你本机通过「抓包工具」捕获后写入 `config.json`，不会上传、不会打包、不会丢失。

## 🌐 支持的网站

- [任意门](https://www.anyanygame.com/turing-test*)（anyanygame.com）
- [更好的图灵测试](https://game.xfcode.top/*)（xfcode.top）—— 完整站点适配器（输入、发送、判定、结算、再次匹配全自动）
- [更好的图灵测试 · 测试站](https://test.xiaofengqwq.com/*)（xiaofengqwq.com，与 xfcode 同源代码）
- [自建网页客户端](http://localhost:8890/*)（AnyAnygame.Wechat.Client，脚本内置适配）
- 其他未知站点自动走通用 CSS / 文本启发式兜底

## 🚀 快速开始（主脚本）

1. 安装浏览器油猴扩展：Chrome / Edge 应用商店搜索安装 Tampermonkey（篡改猴）或 Violentmonkey（暴力猴），并在扩展管理页开启「开发人员模式」；
2. 下载 [Turing AI Free Toolkit V1.5.js](../Turing%20AI%20Free%20Toolkit%20V1.5.js)（或从发行压缩包 `Turing.AI.Toolkit.Windows.amd64.zip` / `Turing.AI.Toolkit.Windows.arm64.zip` / `Turing.AI.Toolkit.Windows.386.zip` 中提取），把 `.js` 文件拖进浏览器窗口自动打开导入页，或新建脚本粘贴全文后 `Ctrl+S` 保存；
3. 打开图灵测试页面 → 单击右下角浮动按钮「TUI」（或按 Insert 键）打开控制面板 → 在 API 配置中填入 OpenAI 兼容接口地址与 Key（默认接入 Agnes 纯公益AI站）→ 打开「自动回复」开始运行。

可选：联网搜索需配合 zip 附带的 OpenSERP 后端服务（解压后双击「启动联网搜索后端服务器.py」，需 Python 3，自动在 127.0.0.1:7070 启动，百度引擎免费无需 Key），脚本内打开「联网搜索」即可。

## 🚀 快速开始（微信客户端）

### 环境要求
- **Windows 10 / 11**，无需安装任何运行时（exe 已内置 Node.js 与 openssl）
- 任意现代浏览器
- **请把 exe 放在可写目录**（桌面 / 文档 / 下载等）：`config.json`、证书等数据写入 exe 同级目录；放在 Program Files 等受保护目录将无法工作
- 若杀毒软件误报：将程序加入白名单（无签名 exe 的常见情况，属正常）

### 第一步：双击 `任意门图灵测试客户端.exe`
主页面出现两个入口：

**🎣 抓包工具（首次必做）**
1. 点击「抓包工具」→ 弹出抓包窗口
2. 点「① 安装证书」（首次；自动生成并信任本地 CA，仅当前用户，免管理员）
3. 点「② 开启代理」（自动设置系统代理，退出自动恢复）
4. 打开微信 PC 端 → 运行《全民图灵测试》小游戏 → 登录 / 进入任意界面
5. 工具自动捕获 → 弹窗提示「抓包成功 + 到期时间」→ 已自动写入凭据

> 抓不到？微信设置 → 网络 → 开启「使用系统代理」；或手动把代理填 `127.0.0.1:8888`。
> 兜底方案：用 Reqable 等工具抓包导出 HAR，在抓包工具里点「兜底：导入 HAR 文件」。

**🚀 服务器**
1. 凭据就绪后点击「启动服务器」→ 后台启动本地代理 → 自动打开浏览器进入游戏
2. 关闭窗口即最小化到**系统托盘**（服务器继续运行）——托盘右键可：显示主窗口 / 打开游戏页面 / **停止服务器** / 退出

界面操作：填昵称 → 开始匹配 → 聊天 → 判定真人/AI → 结算（可返回房间继续聊）→ 再匹配一局。

## 🔁 凭据续期（JWT 约 7 天过期）

到期前控制中心凭据状态条黄色预警、过期红色预警：
1. 打开 exe → 抓包工具 → ② 开启代理
2. 打开微信小游戏任意操作一下
3. 捕获成功自动更新凭据，无需手动改任何文件

## 🧩 脚本联动

主脚本已内置客户端适配（localhost:8890）：自动监听聊天、AI 生成回复自动发送、自动判定、全自动连战（结算自动回到主页并自动开始下一局）；打开 `http://localhost:8890` 后右下角出现「TUI」浮动按钮即脚本已注入。

## 📡 技术说明（客户端抓包逆向结论）

- 服务端微信通道：仅校验三件套凭据（`Authorization` JWT / `X-Visitor-Id` / `X-Turing-Channel`），与 UA / Referer / 来源环境无关
- HTTP API：`register` → `match` → `rooms/{id}/guess`
- **WSS 必须走本地代理**（`/ws`）：服务器会掐断带 Origin 头的 WSS 连接（浏览器强制携带 Origin，实测 0 秒 code=1006 被断）；代理用 Node 无 Origin 连接上游正常
- 消息协议：`match.subscribe → match.update(matched) → room.subscribe → room.update / message.send → room.unsubscribe`
- 结算：`room.result = { actualType, guess, correct, reason, opponentGuess }`
- 断连容错：排队断连自动重排（最多 3 次）、排队 48 秒超时保护、限流自动再试、取消防竞态

## 🔒 连线确认（防误安装）

脚本只对以下网址生效，其余页面不会运行：

```
https://www.anyanygame.com/turing-test*
https://game.xfcode.top/*
https://*.xiaofengqwq.com/*
http://localhost:8890/*
http://127.0.0.1:8890/*
```

## 🛣 开发计划（V1.6）

正在规划中的下一版本（以实际发布为准）：

- ⏱ **延迟范围细化**：回复 / 开局问候 / 分段 / 补充回复 / 广告等各类发送动作的延迟独立可调（开发者模式）
- 🚫 **网址发送防护**：按站点规则拦截网址发送（xfcode / test.xiaofengqwq 系列站点除外）；广告含网址在菜单红标提醒
- 🧯 **重复发送修复**：AI 偶然重复相同内容的根治
- 🎯 **己方消息严格区分**：抢先问候、广告等不再被误认为对方发言
- 📦 **超长广告自动分段**：广告超过 260 字自动分段发送，分段同样遵循延迟范围
- ✏️ **提示词自定义**：开发者模式开启后可编辑各模式与决策提示词
- 🧠 **自主学习**：对局样本学习 + 胜负复盘分析，越打越强
- 👁 **多模态识别**：配置可开启多模态选项，自动识别表情包 / 图片（gif、png、jpg、jpeg 等），精准理解对面发的内容并回应
- ⌨️ **热键悬浮窗**：功能按键绑定（切换 / 按住），赛博风格的可视化热键面板
- 🖥️ **客户端托盘修复**：① 托盘「退出」无法彻底退出的毛病（进程 / 图标残留）；② 显示两个托盘按钮的毛病（疑似程序重复启动，加入单实例互斥，重复启动自动聚焦已有实例）

## ⚠️ 免责声明

**本工具仅供学习交流与开发人员本地测试使用**，请勿将本工具用于任何违反平台规则或国家法律法规的场景，由此产生的一切后果由使用者自行承担。请在合法合规的范围内使用。

> 再次声明：本项目是仅供安全研究、学习和本地测试的开发者工具，不是作弊工具，不支持也不鼓励任何违规使用。
> 请完整阅读 [DISCLAIMER.md](DISCLAIMER.md) 与 [SECURITY.md](SECURITY.md)。

- 匹配会把你的微信账号真实地送入官方匹配队列，**会与其他真实玩家对局**
- 绕过开发者运营开关属于灰色操作，账号存在风险，请自行权衡，仅用于学习交流

---

## 📦 项目结构与发行包

仓库根目录同时保管主脚本、各平台发行目录与官网源码。发行策略：

- **V1.5（本次发行）**：发行 **Windows 压缩包** 与 **微信客户端压缩包**，发布在 GitHub Releases（仓库 `KLP-KULIPA/turing-ai-toolkit`），上传的资产为 `Turing.AI.Toolkit.Windows.amd64.zip` / `Turing.AI.Toolkit.Windows.arm64.zip` / `Turing.AI.Toolkit.Windows.386.zip` / `AnyAnygame.Wechat.Client.zip`；
- **V2.0（后续发行）**：未压缩的平台目录（macOS / Linux / 移动端等）届时统一打包发行。

### V1.5 发行压缩包（GitHub Releases 资产）

| GitHub 资产名 | 本地压缩包 | 适用平台 |
| --- | --- | --- |
| `Turing.AI.Toolkit.Windows.amd64.zip` | `Turing.AI.Toolkit.Windows.amd64.zip` | Windows 10/11 · 64 位（x86_64） |
| `Turing.AI.Toolkit.Windows.arm64.zip` | `Turing.AI.Toolkit.Windows.arm64.zip` | Windows 10/11 · ARM64 |
| `Turing.AI.Toolkit.Windows.386.zip` | `Turing.AI.Toolkit.Windows.386.zip` | Windows 10/11 · 32 位（x86） |
| `AnyAnygame.Wechat.Client.zip` | `AnyAnygame.Wechat.Client.zip` | Windows 10/11 · 自建网页客户端（零依赖 exe） |

> 主脚本压缩包内容：`Turing AI Free Toolkit V1.5.js`、openserp.exe（Windows 搜索后端）、启动联网搜索后端服务器.py、安装说明.txt。
>
> 本地压缩包已按英文命名，直接上传到 Releases 即可。

### amd64 与 arm64 的区别（先看这里再下载）

**amd64 与 arm64 都是 64 位系统**，区别在于 CPU 的指令集架构，两者互不通用，**不能混装**：

| 名称 | 架构 | 常见 CPU |
| --- | --- | --- |
| **amd64**（也叫 x86_64 / x64） | 传统的 Intel / AMD 64 位架构 | Intel Core i3/i5/i7、AMD Ryzen 等绝大多数电脑 |
| **arm64**（也叫 AArch64） | ARM 公司推出的 64 位低功耗架构 | 苹果 M1/M2/M3… 系列、高通骁龙 X（Windows ARM 笔记本）、多数手机平板 |

简单记：**按你电脑上看到的是「Intel/AMD 处理器」还是「ARM / M 系列芯片」来选**。三个平台（Windows / macOS Darwin / Linux）对应 3 组文件夹，辨别方法如下：

#### Windows 用户（对应 `Turing.AI.Toolkit.Windows.xxx/` 目录与 `.zip`）

1. 依次打开：**设置 → 系统 → 关于**（或右键「此电脑」→ 属性）；
2. 看「系统类型」一栏：
   - 显示 **「64 位操作系统，基于 x64 的处理器」** → 下载 **amd64**
   - 显示 **「64 位操作系统，基于 ARM 的处理器」** → 下载 **arm64**
   - 显示 **「32 位操作系统」** → 下载 **386**
3. 更精确：按 `Win + R` 输入 `cmd` 回车，执行命令看结果：
   ```
   echo %PROCESSOR_ARCHITECTURE%
   ```
   - 输出 `AMD64` → 选 **amd64**
   - 输出 `ARM64` → 选 **arm64**
   - 输出 `x86` → 选 **386**

> 提示：绝大多数台式机与普通笔记本（Intel / AMD 处理器）都是 amd64；只有少部分 `Surface Pro X`、骁龙版轻薄本、Mac 上的 Windows（ARM）才是 arm64；386 仅用于 32 位老系统。

#### macOS 用户（怎么选 `Turing.AI.Toolkit.Darwin(MacOS).amd64/` 或 `- arm64/` 文件夹）

1. 点屏幕左上角**苹果菜单 → 关于本机**；
2. 看「芯片 / 处理器」一栏：
   - 显示 **Apple M1 / M2 / M3 / M4…** → 选 **arm64**
   - 显示 **Intel…**（英特尔）→ 选 **amd64**
3. 更精确：打开「终端」，执行：
   ```
   uname -m
   ```
   - 输出 `x86_64` → 选 **amd64**
   - 输出 `arm64` → 选 **arm64**

#### Linux 用户（怎么选 `Turing.AI.Toolkit.Linux.xxx/` 文件夹）

打开终端执行以下命令，按输出选文件夹：

```
uname -m
```

| 输出 | 架构 | 选哪个文件夹 |
| --- | --- | --- |
| `x86_64` | 64 位（Intel/AMD） | `Linux - amd64` |
| `aarch64` | 64 位（ARM） | `Linux - arm64` |
| `i386` / `i686` | 32 位 | `Linux - 386` |

> 不确定时先执行 `uname -m`，不要凭印象下载；选错架构 openserp 无法启动。

### 未压缩平台目录（本次不发行，V2.0 再发行）

| 目录 | 平台 / 架构 | 内容 |
| --- | --- | --- |
| `Turing.AI.Toolkit.Windows.amd64/` | Windows 64 位（x86_64） | 脚本 + openserp.exe + 启动后端 + 安装说明 |
| `Turing.AI.Toolkit.Windows.arm64/` | Windows ARM64 | 同上 |
| `Turing.AI.Toolkit.Windows.386/` | Windows 32 位（x86） | 同上 |
| `Turing.AI.Toolkit.Darwin(MacOS).amd64/` | macOS 64 位（x86_64） | 脚本 + openserp + 启动后端 + 安装说明 |
| `Turing.AI.Toolkit.Darwin(MacOS).arm64/` | macOS Apple Silicon（arm64） | 同上 |
| `Turing.AI.Toolkit.Linux.386/` | Linux 32 位（i386） | 同上 |
| `Turing.AI.Toolkit.Linux.amd64/` | Linux 64 位（x86_64） | 同上 |
| `Turing.AI.Toolkit.Linux.arm64/` | Linux 64 位（arm64） | 同上 |
| `Turing.AI.Toolkit.Mobile/` | 手机 / 平板（Android / iOS） | 仅脚本（移动端版本），不打包 |

### 其他目录

| 目录 | 说明 |
| --- | --- |
| `网页/` | 官方网站源码（本 README.md、index.html、头像），部署到 GitHub Pages |
| `AnyAnygame.Wechat.Client/` | 自建网页客户端发行目录（控制中心 exe + src 源码） |

---

## 🌐 官网源代码（GitHub Pages）

本目录同时是「图灵AI辅助」的官方网站源码，部署到 GitHub Pages 后即：

- 官网：https://klp-kulipa.github.io/turing-ai-toolkit/
- 油猴脚本的「检查更新」数据源（脚本读取页面徽章版本号 `V1.5` 进行比对）

### 目录结构

| 文件 | 说明 |
| --- | --- |
| `index.html` | 官网单页（首页 / 功能 / 演示预览 / 安装教程 / FAQ），版本号显示在页面徽章上 |
| `README.md` | 本文档（产品介绍 + 发行说明 + 网站部署说明） |
| `DISCLAIMER.md` | 完整免责声明 |
| `SECURITY.md` | 安全与合规说明 |
| `苦力怕.KULIPA头像【圆形】.png` | 站点 / GitHub 头像（KULIPA · 苦力怕，圆形） |

> 网页的「下载」按钮指向 GitHub Releases（`KLP-KULIPA/turing-ai-toolkit`）的 `V1.5` tag，无需在站点目录里放压缩包；发布流程见「打包更新流程」。

### 部署方式

将本目录内容推送到 `KLP-KULIPA` 的 GitHub Pages 仓库（仓库名 `KLP-KULIPA.github.io`），站点部署到 `/turing-ai-toolkit/` 子路径：

1. GitHub → 新建仓库（建议仓库名 `KLP-KULIPA.github.io`，私有公开均可）；
2. 将本目录的 `index.html`、`README.md`、头像文件放入仓库的 `turing-ai-toolkit/` 子目录并推送；
3. 仓库 Settings → Pages → Source 选择 `main` 分支 / 根目录（部署后站点位于 https://klp-kulipa.github.io/turing-ai-toolkit/）；
4. 等待几分钟后访问 https://klp-kulipa.github.io/turing-ai-toolkit/ 即可。

### 版本检查机制（重要）

油猴脚本 [Turing AI Free Toolkit V1.5.js](../Turing%20AI%20Free%20Toolkit%20V1.5.js) 启动后会自动请求本站：

```
GET https://klp-kulipa.github.io/turing-ai-toolkit/
```

然后从 HTML 中匹配版本徽章：

```
/<div class="badge">[^<]*<span class="dot"><\/span>V(\d+(?:\.\d+)?)/i
```

同时比对脚本内部 `SCRIPT_VERSION`（当前 `1.5`）与页面 `SCRIPT_VERSION`（`1.5`），一致即视为已是最新；不一致时提示前往官网更新。**发布新版时必须同步更新**：网页 `index.html` 的徽章与 `SCRIPT_VERSION`、脚本 `@version` 与 `SCRIPT_VERSION`。

---

## ✅ 合规声明

1. 本项目**仅限在本地环境(localhost)中运行**，禁止用于任何违反服务条款或法律法规的场景。
2. 本项目是**仅供安全研究、学习和本地测试的开发者工具**，仅限具备技术能力的开发人员用于学习交流。
3. 本项目严格遵守 **GitHub 可接受使用政策**，用户须自行承担使用本软件的一切风险。
