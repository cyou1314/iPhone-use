# 调研笔记：Poki 逆向与地基链先例（2026-11-09）

本文件是自研路线的事实底稿。来源：Poki 开源仓库（JosssphZhou/poki）源码与作者开发交接文档
HANDOFF.md 的审计（审计副本曾放在本地 ~/workspace/poki-audit/）、StikDebug / SideStore 官方仓库说明。

## Poki 的两条线（别混淆）

1. **Mac 版（MCP / Agent，v0.1.1）**：控制器在 Mac 上，走苹果原生开发者通道操作真机——
挂载开发者镜像（DDI）后用其中的 display service 注入触控、RemoteXPC 启停 App；
不需要在手机上装 WDA 替身。眼睛是截图 + 无障碍控件树（带 OCR 补位）。
2. **手机独立版（standalone 分支，Alpha，0.2.x）**：我们要做的形态。见下节。

## 独立版的三个机关（Poki 实现，代码与文档互证）

1. **进程内 testmanagerd**：App 在自身进程内经 testmanagerd（Xcode UI 测试的系统守护进程）
读控件树、点按、打字、启停 App；默认不装 Runner、不监听端口。
（可选 Runner 构建会开 8100 端口且暴露局域网——已知风险点，作者文档明示。）
2. **自环隧道**：App 够不到手机自身的开发者服务，于是用包隧道把发往 10.7.0.1 的流量反射回本机，
触达 RemotePairing（49152）/ lockdown（62078）。蜂窝下系统会拒绝、需 Wi-Fi。
自带隧道版需要 Network Extension 权限（免费团队签不出）；外置版借 Karing 等现成代理 App 提供回环。
3. **装机注入**：Mac 上的安装器（Poki Setup）用用户本人 Apple 账号完成配对、开开发者模式、
签名安装，并把配对文件与 DDI 交给 App。免费号 7 天续签。

脑与护栏：App 内置对话循环（pi），模型 Key 在手机上或 ChatGPT 登录；敏感动作走灵动岛批准卡
（最长等 115 秒），任务结束自动回原 App；远程派活靠自建 Cloudflare relay，无默认地址。
审计印象：Mac 侧代码无埋点 SDK，外发只有模型 API / Telegram / GitHub 更新检查（可关）；
可选的锁屏密码解锁把密码存 Mac 本地（不进模型与日志）——均为作者明示的设计。

## 地基链的独立先例（不依赖 Poki）

- **StikDebug**（StikDebug/StikDebug，iOS 17.4+）：免费号侧载的 App，在手机上开回环隧道、
连 RSD、挂 DDI、启用调试/JIT 会话——证明「回环 → 配对 → DDI → 调试通道」可纯手机跑通。
其底层是 **idevice**（Rust 协议栈：lockdown / RemoteXPC / DDI / DTX），Poki 的 launcher 亦用同系实现。
- **SideStore**：免费 Apple ID 侧载与到期自动续签的标准解（配对文件 + 回环 muxer）。
- **jitterbugpair**：一次性生成配对文件的开源小工具。
- **WebDriverAgent**（Appium）：成熟的 UI 自动化执行器，本项目用作「手」的保底路线 B。

## 与其他同类的一句话区别

- iPhone-use（zhongerxin）：Mac + Codex 经 USB 操作真机，自签 WDA 路线（7 天过期），默认带匿名统计。
- esp32-iphone-agent：7 美元 ESP32 硬件当手（USB HID）、AirPlay 当眼，手机端零安装，但要外挂硬件。
- AIPhoneAgent（wind-peng）：TrollStore 侧装、App 内直接操作，仅限可装 TrollStore 的旧系统版本。

## 脑源池候选（免费额度优先，2026-11-09 初核）

背景：PLAN 第四节定了脑源池+轮换（先烧免费/订阅、最后付费）。本节是候选名单初核，
判定口径同薅羊毛空间：官方口径优先，第三方数字标注来源，未实测的不冒充能用。
入池前每家还要过 PLAN 的一致性小考（同一截图给出合格式动作），本节只管「额度与资格」。

| 候选 | 免费额度（口径） | 看图 | 对用户的判定 | 备注 |
|------|------------------|------|--------------|------|
| 智谱 GLM 免费视觉系列（GLM-4.6V-Flash / GLM-4.1V-Thinking-Flash / GLM-4V-Flash） | 官方文档列为免费模型（docs.bigmodel.cn 免费模型页），0 元调用；限速数值不公开、控制台可查，非按天封顶型 | ✓ 本行即视觉系列；4.6V-Flash 128K 上下文 | **首选**：手机号注册即得 Key（open.bigmodel.cn），国内直连 | GLM-4V-Flash 输出上限小（第三方实测 max_tokens ≤1024），给动作决策够用；免费只覆盖 4.x 代 Flash，5.x 旗舰收费，model id 别写死、防下线 |
| 阿里百炼（通义千问）新用户免费额 | 第三方情报口径：多数模型各约 100 万 Token 免费额、有效期 90 天、快照与最新版额度独立计——**以注册后控制台到账为准** | ✓ Qwen-VL 系列 | **有条件**：国内账号可领，额度与覆盖模型须控制台实读 | 一次性额度（90 天），适合第 1 期调试期烧；过期后从池中降级 |
| Gemini API 免费档（Google AI Studio） | 官方不公布统一限额表；第三方按 AI Studio 限额页汇总：Flash-Lite 约 500 次/天、Flash 约 20 次/天，按项目计、同项目多 Key 不叠加 | ✓ Gemini Flash 系列原生多模态 | **有条件/需实测**：支持地区不含中国大陆，jaycyou21 能否发 Key、手机网络能否直连均未验 | 免费档数据可能被用于改进产品——截图会发过去，敏感界面任务慎用此源（护栏层可按任务排除） |
| ChatGPT 登录（Codex 后端） | 非免费，是 Plus 已付沉没成本；与 Mac 上 Codex 共用同一额度池 | 待验（截图喂入未测） | **实验项**：已在 PLAN 第四节，细节见彼处 | 吃额度快、通道可能被官方收紧，不作主力 |
| GitHub Models 免费档 | 曾有：高阶模型约 50 次/天、低阶约 150 次/天（第三方汇总） | ✓（GPT-4o 等） | **存疑，暂不纳入**：有来源称该服务已于 2026-07-30 退役，与其他来源冲突，未实测 | 若日后实测可用再补入；不为它先做适配 |
| OpenRouter :free 模型、Groq 等 | 未核 | 部分 ✓ | **未核候选**：不列入首批，池子跑通后按需逐个核 | — |

**明确不可用（防再惦记）**：Kaggle AI Models 的 $10/天额度锁死在 Kaggle Benchmarks 环境内，
不能接出当通用 API，无法作脑源；Kaggle GPU 亦无常驻端点供手机调用。

**省流设计（等于扩容，PLAN 已有/已定）**：控件树优先、树够用不发截图；简单步骤用免费 Flash 档、
难步骤才上强模型；任务内粘性一家避免重复热身。同等额度下任务数可差数倍。

**红线（与薅羊毛空间一致）**：池子的加法只在不同家之间做，一家一个正常账号；
同一家多开账号翻倍额度属各家明文禁止，不做。

## 对本项目的结论

- 地基（装机/回环/配对/DDI/调试会话）有现成先例与组件，可拼装。
- 唯一没有傻瓜现成件的是「进程内 UI 自动化会话」（testmanagerd 客户端），Poki 是存在性证明；
本项目先以 WDA 路线保底跑通整机，再攻此环。
- 免费账号的硬约束：7 天续签、组件数受 App ID 配额限制、自带包隧道签不出（走外置回环）。

---

## 第一轮周巡（2026-11-16，基线 2026-11-09 → 本轮）

本轮专题线：① testmanagerd 会话授权机制。下轮：② iOS 27 的 DDI/RSD/开发者模式变化。

### Poki（JosssphZhou/poki）

- main 活跃：0.4.2 Beta（2026-11-15；自动化水印仅任务中显示、#368 提示修复、Poki Setup 0.2.3 说明）；
0.4.0 Beta（2026-11-14，全新界面）；Poki Setup 0.2.4（2026-11-16，Apple 登录验证码修复 #403：
等苹果主动推送的码、一次登录只问一个码）。
- standalone 分支：最新提交 2026-11-11（合入 main 的 OpenCode Go 账号接入 + 读屏截止时间修复 #139、
设置页路由修复 #137）；11-11 之后无新提交。
- HANDOFF.md：最后改动 2026-11-07（基线前），本轮无变化；main 与 standalone 内容一致。
- 真机反馈 issue（本轮新开，均未关闭）：
  - #382（2026-11-15）：iOS 27.3 上 "Disk image outdated"，Runner stopped；Setup 显示 DDI "Ready"
  但任务全失败；iPhone 15 Pro Max / iOS 27.3 / Poki 0.4.1 (1382)；作者关联 #18（Cryptex DDI 不匹配）。
  - #377（2026-11-14）：iOS 27.3 上 Automation 开关弹回，"Enter Your Passcode Next" 提示 22 分钟内
  记了 90+ 次却从不出现，startSession 提前返回；iPhone18,2 / iOS 27.3 (24C5037i) / Xcode 27.2 /
  macOS 27.2.1；回环经 Surge。
  - #320（2026-11-13）：iPhone 17 Pro Max / iOS 27.0.1，五项设置全绿仍开不了"连接自动化"；
  评论中 Proteus-l（iOS 27.3）给出关键证据：手动跑 In-process automation → Prepare 立即报错
  `developer_framework_unavailable`，dlopen XCTestSupport 失败（/System/Developer/… 与
  /private/preboot/Cryptexes/OS/… 均 no such file）；作者回复确认两点：①开关卡在锁屏密码说明页
  （被全屏 Settings 遮住，模拟器复现）；②"Ready" 只校验了下载的镜像文件有效，没校验框架能否加载
  ——DDI 重启后未重挂是可能原因；修复在 #368（0.4.2 Beta：提示浮于 Settings 之上、框架不可用时
  开关旁直接给原因与下一步、"Ready" 改为框架实际可加载才亮）。
  - #208（2026-11-12）：iPhone 14 Pro / iOS 27.2，同样 `developer_framework_unavailable`，
  读屏/操作任务在 connecting → pairing → running 后立即失败；回环经 Shadowrocket（含 10.7.0.1/32 路由）。
  - #375（2026-11-14）：前台 App 不回答辅助功能查询时读屏卡住并阻塞 launch（与 InProcess.m 注释的
  12 分钟 hung 请求呼应）。
- iOS 27.3 已在野（build 24C5037i），Xcode 27.2、macOS 27.2.1 同期出现；Poki 0.4.1 在 27.3 上
DDI 显示过期——DDI 跟进速度是当前最现实的卡点。

### 专题①：testmanagerd 会话授权机制——代码证据收敛

- 证据来源：Poki main @ afc41da8（2026-11-16）源码，`native/ios-app/Poki/Native/RecoveredNative.m`
（SRNativeSession）与 `InProcess.m`。强度：代码（强）+ 作者 issue 回复（作者自述）+ 多人真机
issue（第三方实测）互证。
- 会话建立链（`prepareLocked`，顺序即把关顺序）：
  1. `loadDeveloperFrameworksLocked`：按序 dlopen 四个路径——XCTestSupport、
  lib_TestingInterop.dylib、XCTestCore、XCUIAutomation，均位于 `/System/Developer/…`（即已挂载的
  DDI）；任一失败 → `developer_framework_unavailable`。把关点 1 = **DDI 已挂载且框架能在进程内加载**。
  2. `XCTRunnerDaemonSession sharedSessionPromiseAndImplicitlyInitiateSession:YES`：读共享会话
  promise（3 秒未兑现则放弃等、改开自有会话）。
  3. 自有会话：`connectionForInitiatingSharedSession` +
  `sessionWithConnection:completion:`（注释明示：与 UI 测试 runner 同一套 API）。
  4. `supportsAutomationMode` 与 `enableAutomationModeWithError:`——注释原文："The one call that
  waits on the owner: iOS shows its passcode screen from inside it." 把关点 2 = **每次重启后首次开
  会话须过 iOS 锁屏密码确认**（由 enableAutomationMode 内部触发）。
  5. `registerInvalidationHandler:` 监听会话失效；失效后重开走自有会话路径（共享 promise 会继续
  持有已关闭的会话，再启用会挂一分钟）。
- InProcess.m 注释："the AX client's own elements and snapshot request, which testmanagerd answers
for any process with a session."——即 testmanagerd 对"持有会话的任何进程"回答 AX 请求。
- 结论（收敛 PLAN 附二分歧 A2）：代码中**不存在**"IDE 侧控制连接先声明哪个进程是被授权测试进程"
或"借用已授权进程身份"的步骤；会话由 App 自身进程经 XCTest 私有框架 API 直接建立。真实把关点
是三层可分别验证的检查：①DDI 挂载→框架可 dlopen；②重启后锁屏密码确认（enableAutomationMode 内）；
③testmanagerd 会话有效性（失效监听+重开）。野外故障（#208/#320/#377）全部落在 ① 或 ②，从未
出现 daemon 拒绝客户端身份的报错。Haiku 5.5"借用已授权进程身份"的假说不被代码支持，降级为不成立；
"传输层来源 vs 被声明的测试进程身份"的分歧收敛为：无此类声明步骤。
- 对第 0 期闸 3 的影响：探针应分三段分别记录——框架 dlopen 结果、enableAutomationMode 是否弹出
密码确认、会话读树结果；三段失败对应三种不同的修法（重挂 DDI / 走密码流程 / 查会话），不要混成
一个"会话不通"。

### StikDebug / idevice / SideStore / jitterbugpair

- StikDebug：最新 Release 仍为 3.2.2（2026-11-08，基线前）；2026-11-12 唯一提交是改官网下载链接
指向最新 Release，无功能变化。Release 说明值得记一笔：3.1.11 起 DDI 改走 cryptex 挂载、26.4+
重启后不再掉挂（与 PLAN 闸 2 相关，留待专题②核）。
- idevice（jkcoxson）：最新提交 2026-10-27（基线前），本轮无变化。
- SideStore：最新提交 2026-11-06（SDK 27.2，基线前），本轮无变化。
- jitterbugpair（osy/Jitterbug）：自 2022-09 后无提交，本轮无变化。

### 同类与邻近

- pymobiledevice3：活跃。2026-11-12 新增 `mounter auto-mount --replace`（文档同步把手动挂载
DeveloperDiskImage 改为 auto-mount --replace）；2026-11-15 lockdown 重连修复（_request 里删掉跨循环
死重连、RSD lockdown 断开上报为 disconnect）；2026-11-12 `remote pair` 手动配对设备去重。均属
DDI/配对链路的稳固化，与专题②相关，留待下轮。
- go-ios：2026-11-09～11 新增 HID 能力：经 Indigo digitizer 触摸任意屏幕（含边缘手势，#887）、
按设备实体键（#894）、折叠屏铰链角度（#893）；10-09 已有经 `com.apple.security.cryptexd.remote`
挂载 DDI（#865/#870）。与 testmanagerd 无关，但 HID/Indigo 路线是"手"的另一实现参考。
- WebDriverAgent：最新提交 2026-10-17（基线前），本轮无变化。
- 新入场者（GitHub 搜索，2026-11-09 后创建）：
  - **HyunSeob/iphone-cli（2026-11-10，MIT）**：值得深挖，已克隆静态审计（副本
  ~/workspace/iphone-cli-audit/，HEAD 677c4a62，2026-11-15）。Rust CLI + MCP，Mac/Windows 宿主
  驱动真机，iOS 27+，无 Xcode、无 WDA、手机上不装 App。机制（代码+文档互证）：读屏走 DTX 连
  `com.apple.accessibility.axAuditDaemon.remoteAXService`（`clientNeedsAccessibility:` 要整屏
  dump）；触摸/键盘/按键走 `com.apple.coredevice.displayservice` 屏幕共享媒体会话（只开音频流、
  包丢弃，Xcode 设备窗口同款做法）+ `com.apple.coredevice.hid.universalhidservice`
  （IndigoHidClient）；启停 App 走 `com.apple.coredevice.appservice`；截图走
  `screencaptureservice`；读屏无响应时可杀 axauditd 重启。**全程零 testmanagerd、零 XCUITest**。
  作者自述仅在一台 iPhone 15 Pro Max / iOS 27.0.1（韩文界面）与一台 iPhone19,4 / iOS 27.2
  上验证过。DDI 来源：有 Xcode 用 Xcode 的，否则从 doronz88/DeveloperDiskImage 公开仓库取
  （pinned commit + SHA-256）。
  - 其余新入场多为宿主侧对照组，未深挖：jiefeng-ai/phonectl-mcp（USB MCP）、BO5AMIS/tapwright
  （测自己 App）、ClaudeCode0425/iphone-wifi-agent（WDA + pymobiledevice3 + go-ios 的 Wi-Fi 版）、
  kerimctrl/iphone-use-skill（经 Mac Device Hub，iOS 27+）、gettogo/togo（手机上看 Mac 的 agent）等。
- 新假设（未验证，仅记录）：iphone-cli 证明 iOS 27 的 CoreDevice 服务（axAuditDaemon
remoteAXService + displayservice + universal HID）可在宿主侧完整替代 testmanagerd 做"眼+手"。
这些服务同样走 RSD，而我们的 App 在手机内经回环也能连 RSD——**理论上存在一条不经过
testmanagerd 会话的备用"手"**。是否成立取决于手机内 App 能否打开这些 RSD 服务（权限/沙盒），
须由第 0 期探针另加一个 CoreDevice 探测分支验证；现阶段不改主线、不改闸 3 的一票否决地位，
只列为观察项。

### X 通道（Weir，本轮 2 次搜索均成功返回）

- Weir 余额 4482/5000（2026-12-01 UTC 重置）；搜索 "iphone-cli OR iphone agent testmanagerd"
返回 1 条无关（App Store 过审清单）；搜索 "Poki iPhone agent" 返回 5 条：Poki 0.4.2 Beta
发布热度（openmys 称 0.4.2 攻克自动化水印常驻问题、与 CHANGELOG 一致），无新增技术事实。

### 对 PLAN 的修订（本轮）

- 附二分歧 A2：按上述代码证据收敛，见专题①结论；探针分段记录的要求写入第五节闸 3。
- 风险 #1：补充 iOS 27.3 已在野、Poki 0.4.1 的 DDI 在 27.3 上报过期（#382）；DDI 跟进速度仍是
闸 1/闸 2 的现实卡点。
- 新增观察项：CoreDevice 备用"手"（iphone-cli 路线），列入风险清单观察项，待第 0 期探针分支验证。

---

## 第二轮周巡（2026-11-23，基线 2026-11-16 → 本轮）

本轮专题线：② iOS 27 的 DDI 个性化/RSD/开发者模式变化。下轮：③ 脑源池免费额度变动复核。
再下轮：④ 回环隧道提供者与 VPN 槽冲突实证 → 回到 ①。

### Poki（JosssphZhou/poki）

- main HEAD dbe4ff6c（2026-11-22，AGENTS.md 改写）；0.5.0 已开版（7ab0046c）。
- Release：0.4.5 Beta 1（2026-11-19，Apple 登录 -22411 修复：一次登录最多问一个 2FA 码、401 -22411
不再当作限流）、0.4.6 Beta 1（2026-11-20，Duo 全面支持、Poki Anytime 语音、按名开 App、记忆/技能页）、
0.4.7 Beta 1（2026-11-21，热修复：iPhone 17 Pro 启动崩溃 #587/#554、Photos 误触发付款提示 #576、
灵动岛批准卡折叠后仍可点 #594）；Poki Setup 0.2.8 Beta（2026-11-20，安装时挂载 DDI，见 #509）。
- standalone 分支：自 2026-11-11（6e9c858）后**无新提交**，连续两周无动静。
- HANDOFF.md：最后改动仍为 2026-11-07，本轮无变化。
- 真机反馈 issue（本轮新开 34 条，挑与地基链相关的）：
  - #681（2026-11-22）：0.4.7 上 Live Activity 一点就抢回前台并立刻关会话；同日 10.7.0.1 环回
  约 9 小时不应答（13:38–22:37，developer service: off，49152 找不到），晚间自行恢复；Apple 登录
  -22411 再次出现（0.4.5 已修过一轮，仍复发）。
  - #673（2026-11-22）：iOS 27.3 / Poki 0.4.1 上自动化会话永不结束，"Automation Running" 浮层常驻，
  只能强退 App；作者文案称 iOS 27 应走 native services（无 XCTest 会话、无浮层），但该机上回落到
  XCTest 且空闲不关闭。会话生命周期在 27.3 上有问题。
  - #672/#554（2026-11-20/22）：0.4.4–0.4.7 在 iOS 27.2.1/27.3 的 iPhone17,1 / iPhone18,3 上
  启动即 scene-create watchdog 崩溃（0x8BADF00D）。
  - #528（2026-11-19）：iPhone 重启后只开 Clash Mi，Poki 卡在回环检查（10.7.0.1:62078/49152 超时，
  gVisor，clash.meta/1.19.33）；切 LocalDevVPN 后 Automation 就绪，再切回 Clash Mi 可正常跑任务。
  作者已把该流程写进官方指南（见下）。
  - #572（2026-11-20）：折叠的 iPhone Duo 盖板屏上 WeChat 能开，但截图与点按报
  `target_display_unavailable`；展开内屏则正常。#472（已关）：27.3 内屏激活时截到黑色的盖板尺寸图。
  - #617（2026-11-21，作者 ZhehanZhang）：Poki Setup for Windows 在 iOS 18 手机上拿不到 DDI
  （见专题②）。
  - #656/#654（2026-11-22）：虚拟外接显示器原型与 UnrestrictMirroring 调研（见专题②）。
  - #621：iPhone relay 间歇报 offline，开关 Remote Connection 可恢复。

### 专题②：iOS 27 的 DDI 个性化/RSD/开发者模式变化

证据强度：代码（强）+ 作者自述/官方文档（中）+ 多人真机 issue（第三方实测）互证。

1. **iOS 27.2+ 的 RSD 新规矩——多条隧道必须用同一个 host UUID**（idevice e5645743，2026-11-16，
代码注释原文："On iOS 27.2+ every connection to a tunnel must present the same UUID, or the device
tears down the services it advertised to all of them."）。含义：同一设备上若开多条 RSD 隧道
（lockdown、DDI 挂载、testmanagerd 各走一条），须共享同一个 host UUID，否则设备会撤掉已通告的服务。
对 PLAN 的影响：回环链设计须把 UUID 统一写死为一条，不可每连接随机生成——这是新增的设计约束，
不是推翻（见 PLAN 修订）。
2. **TSS 个性化请求收紧**（idevice 2dca9a58，2026-11-16）：为拿 timestamp 响应，TSS 请求须补
`Ap,OSEnvironment`、一批安全版本键（`Ap,CurrentOSSecurityVersion`、`Ap,MinimumOSSecurityVersion`、
`Ap,EnableGroundhog` 等）及 build identity 的 `RequestManifestProperties`。DDI 个性化签发链路仍在
跟进苹果侧的请求字段变化。
3. **DDI 挂载分两条路，按 iOS 版本分流**（Poki #617，2026-11-21，作者自述+代码）：
26.4+ 走个性化镜像经 CoreDevice 隧道挂载（cryptex 路径）；26.4 之前（含 iOS 18）走 lockdown over USB
的 image mounter，且挂载要两份配对记录同时有效（Setup 保存的与 USB 服务持有的，Windows 上可能不一致，
任一被拒即挂载失败）。修复分支 `dev/windows-ios18-ddi` 改为先走 CoreDevice 隧道挂载，USB 只作回退；
另修了 seeded 镜像残留 cryptex 文件导致 Poki 拒收的问题。PLAN 闸 1"获取与挂载分开验"得到佐证，
且须注意：我们的测试机是 iOS 27，走的是 cryptex 路径，Windows 双记录坑与我们无关（Mac 路线）。
4. **iOS 27+ 的"手"主路已是 CoreDevice displayservice**（Poki #654，2026-11-22，作者自述）：
Poki 在 iOS 27+ 经 `com.apple.coredevice.displayservice` 驱机（与 Xcode 27 Device Hub 同协议），
iOS 18–26 才回落到 XCTest 会话（带锁屏密码确认与自动化水印）。displayservice 媒体会话打开期间
相机不可用，Poki 只在输入时短开。UnrestrictMirroring（越狱 tweak，不可用）钩子显示：原生
iOS 18–26 拒绝 Device Hub 共享可能源于 dtremotedisplayd 里的生产熔丝（SigningFuse）检查——
解释了版本分流的成因，但越狱方案不在本项目范围。
5. **虚拟外接显示器路线已探、被 entitlement 挡住**（Poki #656，2026-11-22，作者实测）：
iPhone 17 Pro / iOS 27.3 上经 CoreDevice 申请虚拟外接显示器，显示器分配成功（黑屏、独立 uniqueId），
但 `devicectl device process launch --display` 被 SpringBoard 拒绝：缺
`com.apple.springboard.display-lookup` entitlement，DDI 里的 daemon（dtappserviced、testmanagerd
均已查）都不持有。结论："后台跑任务、机主照常用手机"这条路目前无合法 entitlement 可走，暂封存。
6. **会话生命周期在 27.3 上有毛病**（#673）：native services 路径本应无 XCTest 会话、无浮层，
但 0.4.1 在 27.3 上回落到 XCTest 且空闲不关闭；结合 #382（0.4.1 在 27.3 上 DDI 报过期）与
#672/#554（27.2.1/27.3 启动 watchdog 崩溃），27.3 目前是 Poki 的问题集中版本，DDI 跟进仍是卡点。
7. **重启恢复已有作者确认的标准流程**（Poki docs/iphone-guide.md，f294e495，2026-11-21）：
重启后只开 Clash Mi 会卡回环检查，标准做法是先用 LocalDevVPN 启动一次自动化（等日志出现
`[runner] running`），保持 Automation 开着，再断开 LocalDevVPN 切回 Clash Mi。PLAN 闸 2 的
重启恢复应按此流程验。
8. **iPhone Duo（折叠屏）成为新变量**：iOS 27.2.1 上的新机型，盖板/内屏两块屏；Poki 0.4.6
做了全面支持，但盖板屏仍有 `target_display_unavailable`（#572）。我们的测试机是 iPhone 17 Pro
（单屏），Duo 问题不直接影响，但"当前显示屏判定"是执行器必须处理的已知坑。
9. **StikDebug 的配对/连接恢复指引**（f83299ce，2026-11-17，代码+测试）：只加了诊断文案与测试，
无 DDI/RSD 功能变化；另新增按 PID 选 JIT 进程（6299f39）与 Andarium 支持（63713ea2）。
idevice 另有一处配对证书有效期 bug 修复（327414d7：证书寿命算式多了 `*12*31`，约 372 倍，改为 9 年）。

### StikDebug / idevice / SideStore / jitterbugpair

- StikDebug：最新 Release 仍为 3.2.2（2026-11-08）；本轮提交均为小修（见专题②第 9 条），无地基链功能变化。
- idevice（jkcoxson）：本轮恢复活跃，HEAD c8fff6d7（2026-11-22，FFI adapter stream 所有权）；
  关键提交见专题②第 1、2 条及证书修复。
- SideStore：最新提交仍为 4f3d18d9（2026-11-06），本轮无变化。
- jitterbugpair：自 2022-09 后无提交，本轮无变化。

### 同类与邻近

- pymobiledevice3：活跃。2026-11-22 accessibilityaudit 解码 `AXAuditNode_v1` 层级节点
（CoreDevice AX 读屏路径的补强）；同日 crash stackshot/spindump 进程与线程列表。
- go-ios：继续折叠屏 HID 工作：读铰链角度并保持（#898，2026-11-20）、转屏方向（#895）；
  均围绕 iPhone Duo，与 testmanagerd 无关。
- WebDriverAgent：**已搬家**。appium/WebDriverAgent README（30aac069，2026-11-20）声明开发已迁入
appium/appium-ios monorepo（packages/webdriveragent），旧仓只留预编译下载。观察名单指针已更新。
- iphone-cli（HyunSeob）：一周内 0.1.7→0.1.10（2026-11-23）：`iph record` 录屏存 MP4（#11）、
`iph syslog` 读机日志（#9）、Duo 当前显示屏触摸（#12）。仍为宿主侧工具，不改其对照组定位。
- 新入场者（GitHub 搜索，2026-11-16 后创建，63 条命中里筛出 6 个）：
  - safwanadnan/iphoneuse（2026-11-18）：Mac 宿主侧，brew 一键装、WDA 助手 App，MCP 接入；
    宿主指挥版对照组，未深挖。
  - zerosettle/physical-iphone-use（2026-11-17）：纯技能文档（AXe/MCP），无新机制。
  - deemo0666/pallas-iphone-runner（2026-11-19）：XCTest Runner + Mac USB 安装器，Pallas Agent 的
    手机端组件，DSH 插件联动；宿主侧路线。
  - 0bull/iphone-mcp（2026-11-17）：托管真机（云端手机），与"自己手机独立跑"无关。
  - Zeptap/iphone-use（2026-11-19）：Agent skills 文档合集。
  - 其余多为 PWA/实验仓库。本轮无新的**手机独立版**入场者；未发现值得克隆深挖的新项目。

### X 通道（Weir，本轮 2 次搜索均成功返回）

- Weir 余额 4000/5000（2026-12-01 UTC 重置）；两次搜索："Poki iPhone agent"（5 条）与
"iPhone Duo OR iphone-cli OR testmanagerd"（5 条）。内容为 Poki 榜单提及与 iPhone Duo 零售闲聊，
无新增技术事实。

### 对 PLAN 的修订（本轮）

- §三/§六风险 #9：新增约束——iOS 27.2+ 上多条 RSD 隧道必须共享同一 host UUID，否则设备撤掉
  已通告的服务（idevice e5645743）。回环链实现时 UUID 须统一，不可每连接随机。
- §五闸 2：补充作者确认的重启恢复标准流程——先用 LocalDevVPN 启动一次自动化（见日志
  `[runner] running`），再切回日常 VPN（Poki iphone-guide.md，f294e495）。
- 观察项 #11（CoreDevice 备用"手"）：升级——Poki 在 iOS 27+ 已把 displayservice 作主驱机路，
  XCTest 只是 26 及以下回退；虚拟外接显示器后台路线已探明被 entitlement 挡住，暂封存。
  闸 3 的一票否决地位不变。
- 未推翻的旧结论：DDI 跟进速度仍是闸 1/闸 2 的现实卡点（27.3 上多发问题）；standalone 分支
  连续两周无提交，独立版仍停在 0.2.x/11-11 状态。


### 用户点名线索（2026-11-23）

- HyunSeob/iphone-cli（https://github.com/HyunSeob/iphone-cli）：Rust 写的工具，号称让 AI agent 通过无障碍树（accessibility tree）读取并操作真机 iPhone 上的任意 App，不用 XCUITest、不用 WebDriverAgent、不用 Xcode 签名。2026-11-23 在 Upstar iOS 周榜排第 15（七天 +78 星、总星 119）。
- 用户原话「15 你记下，对我后续研究有用」——列为正式调研线索，后续轮次需核：①它与本笔记此前「CoreDevice 备用手（iphone-cli 路线）」观察项是不是同一个东西；②无障碍树路线在 iOS 27 上的实际能力边界（读屏之外能不能点/输入、要不要配对/隧道、是否依赖开发者模式）；③与 PLAN 主线（testmanagerd 最小执行器）的关系——是备选「手」还是可借鉴的读屏层。

### Pack 合并设想（2026-11-23，用户定性：仅调研）

- 用户由 fish-pocket（DSH iOS 版，对接 iCTRL 操控手机）提出「操控手机的 App 与 Pack 合并」，随即定性：现在只是 iPhone-use 前期调研，不立项。合并计划草案在 `~/workspace/pack/PACK-PHONE-MERGE-PLAN-DRAFT.md`，封存待 12-09 收官复盘。
- 对本调研的增量事实：fish-pocket 的操控层是第三方 iCTRL（侧载+开发者模式+首次设置用电脑），其安全口径（敏感动作先问、双音量键急停、只本机监听）可列为「手」的现成参照件，与 testmanagerd 主线、iphone-cli 无障碍树路线并列比较。
