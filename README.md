# 开发全流程 · 从环境侦查到代码提交

> 给 agent／自动化工作写代码的全流程：**环境先侦查 → 先落计划 → 快速验证 → 测试先行 → 迭代式脚本 → 系统化调试 → 预提交审查 → 推送排障**。
> 全套方法论 + 14 份调试工具手册，**纯文档仓**（配方以脚本片段形式写在正文里）。

## 这是什么

一套**踩过坑才写下来**的开发纪律。最贵的三条：

1. **别凭印象动手**——要新增／改一类东西，先把它的机制查到源头做实证（源码／官方文档／原始数据）。机制清了多少，坑就少多少。
2. **「顺手」就真是顺手**——被要求「顺手验证一下／顺手弄一个」时，先交一版能跑的、先把结论给出去；重构、加缓存这类自加项排后面，交完第一版再问要不要做。别把「顺手」做成长工程。
3. **工具出错先自检**——工具报错不跳步。先查参数格式、编码、凭据、路径。跳过这步直接绕路（换工具／加代理）是错误反应。

还有一条交付纪律：**长任务先落计划**（阶段 · 产出 · 验收 · 卡点），**验收不过不进下一阶段**。

## 工作流

| 阶段 | 做什么 | 参考 |
|:----|:------|:-----|
| 🔎 环境侦查 | 动手前摸清 CLI／Python 包／API 可用性 | `references/environment-recon.md` |
| 📋 写计划 | 多步骤任务先写计划再动手 | `references/plan.md` |
| 🧪 Spike 验证 | 不确定能不能行的 → 快速原型验证 | `references/spike.md` |
| 🔴 TDD 红绿重构 | 先写测试再写代码 | `references/tdd.md` |
| 🔄 迭代式脚本 | 处理真实数据的脚本：写了就跑、暴露问题就修 | `references/iterative-script-dev.md` |
| ✂️ 脚本化减法 | 盘点哪些重复劳动可抽脚本；候选判断 + 实例清单 | `references/scriptification-audit-method.md` |
| 🛠️ CLI 落地踩坑 | `--help` 处理／BrokenPipe／输出按调用者照抄设计／子脚本伪交互／报告指标口径分层 | `references/cli-authoring-pitfalls.md` |
| ✂️ 脚本膨胀拆分 | 单文件 >400 行怎么拆；薄壳入口 + 模块包；多 CLI 按需求域分入口 | `references/script-splitting-pattern.md` |
| 🐛 系统化调试 | 找根因，不下随机修（＋14 份工具手册：pdb／debugpy／CDP／heap snapshot） | `references/debug.md` |
| 🔒 预提交审查 | 安全扫描 + 回归 + 独立审查 | `references/code-review.md` |
| 🔧 API 工具报错排查 | 报错先查调用方式再绕路；端点池并发探测 + 缓存 + 错误码分类 | `references/api-tool-troubleshooting.md` |
| 🧹 代码清理 | 四路并行审查找复用／质量／效率／深度问题 | `references/simplify-code.md` |
| 🚚 git 推送排障 | no upstream 自愈／TLS 断连处理／推送要有界 | `references/git-push-troubleshooting.md` |
| 🚦 提交门禁 | 把检查装到 git 上（pre-commit 钩子） | `references/commit-gates.md` |
| ✅ 同步与验收 | 跨档案批量同步后逐文件回读；批处理产物核「每一段都在」；假绿的三种形态 | `references/sync-and-verification.md` |

### 四档速查

| 你要做的事 | 打开 |
|:--|:--|
| **怎么开工**：长任务先落计划／先摸机制再动产出／方向题先答别先侦查／逐个来／发现问题即时修 | `references/working-style.md` |
| **要不要做成工具、CLI 怎么改**：一脚本一职责／多步 ad-hoc → 定版 CLI／后台哨兵失败必须非零退出 | `references/cli-and-scripting.md` |
| **代码与工具坑**：bash 内联反引号／补丁锚点／`json.dumps(indent=2)` 重排手写 JSON | `references/cli-authoring-pitfalls.md` |
| **同步·迁移·验收**：状态类工具先沙盒／判「哪边改了」比上游 ref／对拍自测／验收工具自己的前置失败必须计入退出码 | `references/sync-and-verification.md` |

## 目录

| 路径 | 内容 |
|:---|:---|
| `SKILL.md` | 入口：使用哲学 · 工作流速查 · 四档速查 |
| `references/working-style.md` | 怎么开工（计划四件套、先摸机制、方向题先答…） |
| `references/plan.md` | 计划书写法（阶段 · 产出 · 验收 · 卡点） |
| `references/spike.md` / `tdd.md` | 快速验证 / 红绿重构 |
| `references/iterative-script-dev.md` | 迭代式脚本开发（写主干 → 跑真实数据 → 修 → 再跑） |
| `references/scriptification-audit-method.md` | 「哪些重复劳动该抽成脚本」的判据与实例清单 |
| `references/cli-authoring-pitfalls.md` | CLI 落地踩坑全集（含 `--help`／BrokenPipe／输出设计） |
| `references/cli-and-scripting.md` | 什么时候该做成 CLI、怎么改 |
| `references/script-splitting-pattern.md` | 脚本膨胀后的拆分模式（数据／增强／逻辑三块） |
| `references/debug.md` + `references/debug-tools/` | 系统化调试 + 14 份手册（Python／Node／CDP／堆快照） |
| `references/code-review.md` / `simplify-code.md` | 预提交审查 / 四路清理 |
| `references/api-tool-troubleshooting.md` | API 工具报错排查 |
| `references/environment-recon.md` | 环境侦查（CLI／包／凭据可用性） |
| `references/git-push-troubleshooting.md` / `commit-gates.md` | 推送排障 / 提交门禁 |
| `references/sync-and-verification.md` | 同步与验收（回读、对拍、假绿） |
| `references/skill-authoring.md` | SKILL.md 的 frontmatter 与结构规范 |

## 读者须知

文中出现的 `<数据根>`＝放 `skills/` 的那层目录、`bin/`＝数据根下的命令目录，都是**作者环境的布局**；表里出现的 skill 名（`record-officer-daily`／`internet-memes-reference` 这类）是作者环境技能库的成员，**未单独公开**。判据、坑与验证方法与这些路径无关——照自己的项目替换即可，实测数字读作「某次实战样本」。

## 姊妹仓库

- [liya-delegation-and-verification](https://github.com/feverZHONG/liya-delegation-and-verification) —— 委派与验收：给子代理写任务书、并行隔离、把「自报」验成事实（本仓「独立审查」那一步的正本）
- [liya-subtraction-skill](https://github.com/feverZHONG/liya-subtraction-skill) —— 技能库做减法：减法优先、去重、归档、拆薄
- [liya-persona-authoring](https://github.com/feverZHONG/liya-persona-authoring) —— 给 AI agent 写它自己的身份文件（SOUL.md）
- [liya-prose-quality-metrics](https://github.com/feverZHONG/liya-prose-quality-metrics) · [liya-story-revision-plan](https://github.com/feverZHONG/liya-story-revision-plan) · [liya-corpus-line-mining](https://github.com/feverZHONG/liya-corpus-line-mining) —— 写作三件（量尺 / 修订方案 / 语料挖句）
- [liya-sillytavern-cards](https://github.com/feverZHONG/liya-sillytavern-cards) · [liya-tavern-card-refinement](https://github.com/feverZHONG/liya-tavern-card-refinement) · [liya-sillytavern-worldbook](https://github.com/feverZHONG/liya-sillytavern-worldbook) —— 酒馆角色卡三件（写卡 / 精修 / 世界书）
- [liya-vision-recognition-traps](https://github.com/feverZHONG/liya-vision-recognition-traps) —— 视觉模型识图陷阱
- [liya-chat-game-referee](https://github.com/feverZHONG/liya-chat-game-referee) · [liya-spy-game](https://github.com/feverZHONG/liya-spy-game) · [liya-sea-turtle-soup](https://github.com/feverZHONG/liya-sea-turtle-soup) —— 聊天里能玩的三件
- [liya-ruozhiba-wordbank](https://github.com/feverZHONG/liya-ruozhiba-wordbank) —— 弱智吧题防御手册
- [liya-subtitle-proofreading](https://github.com/feverZHONG/liya-subtitle-proofreading) —— 字幕校对/重建/外挂 SRT
- [liya-news-verification](https://github.com/feverZHONG/liya-news-verification) —— 验证伞：轻量核查／交付前多源验证／链接危险识别／厂商官宣核实／链接考古（含 link_check 工具族）
- [liya-knowledge-persistence](https://github.com/feverZHONG/liya-knowledge-persistence) —— 知识持久化：信息该放记忆层／文件／技能库的分层规范（附记录完整性、语料减法、归档模式）

## 提思路 / 提修正

- 你那边的调试配方、CLI 踩坑、验收手段 → 开 [Issue](https://github.com/feverZHONG/liya-dev-workflow/issues)，把「什么场景、错在哪、怎么修的」写清
- 想直接改 → Fork + PR

## 许可

**双许可**——文档与代码分开：

- **代码**（`scripts/` 下的文件）：**MIT** —— 拿去用、改、再发，保留版权声明即可。
- **文档**（`SKILL.md`、`references/`、本 README 的正文）：**[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)** —— 可以自由使用、改编、连商用都行，**但要署名**（莉娅 / [@feverZHONG](https://github.com/feverZHONG)）并注明来源。

两份全文：`LICENSE`（MIT）／`LICENSE-DOCS`（CC BY 4.0）。

---

*莉娅（[@feverZHONG](https://github.com/feverZHONG)）· 宇宙美好记录官*
