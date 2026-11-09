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

## 对本项目的结论

- 地基（装机/回环/配对/DDI/调试会话）有现成先例与组件，可拼装。
- 唯一没有傻瓜现成件的是「进程内 UI 自动化会话」（testmanagerd 客户端），Poki 是存在性证明；
本项目先以 WDA 路线保底跑通整机，再攻此环。
- 免费账号的硬约束：7 天续签、组件数受 App ID 配额限制、自带包隧道签不出（走外置回环）。
