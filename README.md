# iPhone-use（仓库名；产品名待定）

一个装在 iPhone 上的 App，让大模型**在手机上自己操作手机**：你说一句话，它自己开 App、看屏幕、点按、打字，把事办完；敏感动作停下来等你本人批准。

- 日常运行纯手机，不依赖电脑；电脑只在第一次装机时用一次
- 不越狱、不用付费开发者账号（免费 Apple ID 侧载）、无额外硬件
- 大脑用云端视觉模型 API，眼睛和手在手机里（开发者服务通道）

## 状态

**设计阶段，未开工。** 完整方案见 [PLAN.md](PLAN.md)，前期逆向调研见 [docs/research-notes.md](docs/research-notes.md)。

已定约束：只做手机独立版；零付费项；测试机 iPhone 17 Pro / iOS 27.0。

## 路线一句话

第 0 期先验地基（SideStore + StikDebug 的回环/RSD/DDI 链）→ 第 1 期 WDA 执行器跑通整机 →
第 2 期换 testmanagerd 直连 → 第 3 期补护栏与体验。不过一期，停一期，不硬推。

## 红线

- 配对文件（*.mobiledevicepairing）、API Key、Apple 账号信息**永不入库**（.gitignore 已挡，提交前仍要自查）
- 借鉴的开源项目（Poki 为 AGPL 等）只研读原理；引用代码前先过许可证
- 结论以真机实测为准，文档里分清与
