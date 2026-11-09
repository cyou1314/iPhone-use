# 前期调研计划（一个月，2026-11-09 → 2026-12-09）

> 用户 2026-11-09 指示：针对「手机独立版 iPhone Agent」这一方面、Poki 及类似项目，
> 持续做调研了解与逆向，把前期调研做成熟。本文件是这一个月的调研章程。

## 边界（先说清）

- 只做**前期调研**：读代码与文档、跟版本与 issue、核事实、做静态逆向审计、修订 PLAN。
- **不开发、不安装、不动用户设备**；第 0 期实机验证不在本计划内，等用户另行发话。
- AGPL 项目（Poki）只研读、不抄码；凭证、配对文件永不入仓库（.gitignore 已挡）。
- 结论以实测/代码为准；只有作者自述的明确标注「自述」；查不到就写查不到。

## 观察名单（watchlist）

核心对象（每周必看）：
- **Poki**（JosssphZhou/poki）：main 与 standalone 分支的提交、Release、HANDOFF 更新、issue 里真机反馈（尤其独立版真机整程、ChatGPT 登录回调、testmanagerd 会话相关）。
- **StikDebug / idevice**：iOS 27 适配、DDI 挂载与 RSD 链的变化、新版本发布。
- **SideStore / jitterbugpair**：续签机制与配对流程变化。

同类与邻近（每周扫一遍，有动静深挖）：
- zhongerxin/iPhone-use（Mac 指挥版，对照组）、wind-peng/AIPhoneAgent、esp32-iphone-agent。
- WebDriverAgent / go-ios / pymobiledevice3：testmanagerd、DTX、processcontrol 的新进展。
- 新入场者：GitHub 搜索关键词（iphone agent、ios agent、testmanagerd、on-device automation 等）与 X 上的同类发布（Weir 通道，读不到如实记录）。

专题线（贯穿全程，逐周推进）：
1. **生死线专题**：免费侧载 App 进程内 testmanagerd 会话授权机制——把 Poki 实现、go-ios/pymobiledevice3 的会话建立代码、各家 issue 讨论对齐成一份机制说明，目标是把 PLAN 附二里「会话把关点到底在哪」这个分歧尽量用代码证据收敛。
2. **iOS 27 专题**：DDI 个性化签发与镜像生态、RSD/配对/开发者模式在 27.0 的任何变化（开发者文档、工具仓适配提交、社区实测帖）。
3. **脑源池专题**：候选免费额度的变动（智谱免费视觉系列的模型上下线、百炼免费额政策、Gemini 免费档限额与地区），有变动就更新 research-notes 的候选表。
4. **回环专题**：回环隧道由谁提供、VPN 槽冲突的更多实证（StikDebug/SideStore/Karing 的文档与 issue）。

## 节奏与产出

- **每周一轮**（约周一上午）：扫观察名单 + 推进一条专题线；有实质发现就更新
  `docs/research-notes.md`（带日期）并推仓库；结论若推翻 PLAN 假设，同步修订 PLAN 并在周报中标红说明。
- **周报**：每周一轮结束后给用户一条简短汇总（有什么新发现、哪条结论变了）；无实质变化时只写日志、不发长文。
- **收官（2026-12-09）**：出「成熟版前期调研结论」——四未知点各自的证据等级升降、专题 1 的机制定论、
  PLAN 需要改的清单、以及第 0 期实机验证的最终建议（做/不做/怎么做最便宜）；同日停掉每周任务。

## 已有家底（起点，不重做）

- Poki 双线逆向审计（2026-11-09，副本在审计方本地）。
- PLAN v2 + 六家模型评审纪要（PLAN 附二）。
- 脑源池候选初核（research-notes「脑源池候选」节）。
- StikDebug 地基链先例结论（research-notes「地基链的独立先例」节）。
