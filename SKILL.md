---
name: dev-workflow
tier: T1  # T分级: T2=直接做 / T1=先请示 / T0=一律拒
description: 开发全流程——从环境侦查到代码提交，含迭代式脚本开发方法论。
---

# dev-workflow · 开发全流程

> 从环境侦查到代码提交的完整工作流。
>
> **读者须知（路径与工具）**：文中的 `<数据根>`＝放 `skills/` 的那层目录（作者环境默认在固定位置），
> `bin/` 指数据根下的命令目录，`skills/<name>/scripts/` 指各技能自带的脚本；表里出现的 skill 名
> （`record-officer-daily`／`internet-memes-reference` 这类）是**作者环境技能库的成员，未单独公开**。
> 判据、坑与验证方法与这些路径无关——照自己的项目替换即可；实测数字读作「某次实战样本」。

## ⚠️ 使用哲学

**这是一套自用的工具箱，不是交付给用户的汇报材料。** 用起来不列清单、整理不自吹——做了就行，不用每步都展示。顺手最重要。

- **工具是自己的手，不是递给用户的遥控器**：CLI/skill/脚本是为了自己查得更快更准，不是交付给用户自己敲。汇报只给结论，绝不写「命令给你，你自己也能查」——工具一旦被当成交付物，活就推回给用户了
- **「顺手」就真是顺手**：被要求「顺手验证一下／顺手弄一个」时，先交一版能跑的、先把结论给出去；性能优化、重构、加缓存这类自加项排在后面，交完第一版再问要不要做——别把「顺手」做成长工程
- **别自造需求**：答完「要不要做 X」就收住，别顺势推销下一个优化项——用户会反问「有没有那个必要」。任何优化先**算清账**：省下的量级 vs 动刀的风险（实测：给技能索引 description 瘦身省 ~145 token／轮，代价是可能删掉一个触发词→该用的时候想不起来——这笔账是亏的）。**没症状就别动刀。**

## 什么时候用

任何涉及开发的任务——新功能、修 bug、重构、代码审查、环境验证。

## 工作流速查

| 阶段 | 做什么 | 参考 |
|:----|:------|:-----|
| 🔎 环境侦查 | 动手前摸清 CLI/Python 包/API 可用性 | `references/environment-recon.md` |
| 📋 写计划 | 多步骤任务先写计划再动手 | `references/plan.md` |
| 🧪 Spike 验证 | 不确定能不能行的→快速原型验证 | `references/spike.md` |
| 🔴 TDD 红绿重构 | 先写测试再写代码 | `references/tdd.md` |
| 🔄 迭代式脚本 | 处理真实数据的脚本：写了就跑、暴露问题就修 | `references/iterative-script-dev.md` |
| ✂️ 脚本化减法 | 盘点技能库哪些重复劳动可抽脚本；候选判断+实例清单 | `references/scriptification-audit-method.md` |
| 🛠️ CLI 落地踩坑 | 建 CLI 的工程细节：--help 处理/BrokenPipe/HTML去标签/gitignore判断/验收清单 + symlink 覆盖坑/子命令黑名单闸门 + 输出按调用者照抄设计（别让调用者现算）+ 散装脚本收「统一入口」（只分发不合并、数据根参数化、换空根实跑验收）+ 子脚本伪交互提示（父入口给 DEVNULL 时 input() 每次吐一行垃圾）+ 报告指标口径分层（模拟数/实测数分写） | `references/cli-authoring-pitfalls.md` |
| ✂️ 脚本膨胀拆分 | 单文件>400行时拆三模块：数据/可选增强/逻辑——rules+enhance+主文件；CLI 多子命令改拆「薄壳入口+模块包」（软链 sys.path/依赖方向/注册器与 lint 闭环）；需求域分化再拆多 CLI（查询/管理分入口）；归档型 skill 的 INDEX 按类分节+自动追加归节 | `references/script-splitting-pattern.md` |
| 🐛 系统化调试 | 找根因，不下随机修（方法论 ＋ 14 份工具手册：pdb／debugpy／CDP／heap snapshot） | `references/debug.md` |
| 🔒 预提交审查 | 安全扫描+回归+独立审查 | `references/code-review.md` |
| 🔧 API 工具报错排查 | 报错先查调用方式再绕路；端点池慢/半死的并发探测+缓存+错误码分类 | `references/api-tool-troubleshooting.md` |
| 🧹 代码清理 | 4路并行审查找复用/质量/效率/深度问题 | `references/simplify-code.md` |
| 🚚 git 推送排障 | no upstream 自愈/GnuTLS 断连处理 | `references/git-push-troubleshooting.md` |

## 子领域

### 环境侦查

动手前确认 CLI 在 PATH、Python 包可 import、API 端点可达。详见 `references/environment-recon.md`。

### 方法论原则

- **工具出错先自检** — 工具报错不跳步。先查参数格式、编码、凭据、路径。跳过这步直接绕路（委托/换工具）是错误反应。见 `references/api-tool-troubleshooting.md`
- **TDD** — 先测试后代码。RED→GREEN→REFACTOR
- **复用别人的脚本，别 import 它的顶层代码** — 借内部函数前先看有没有模块级副作用（裸 `asyncio.run(main())`、顶层发送/写盘）。2026-10-07 实测：为借 `get_token()` import 一个测试脚本，一 import 就跑完整轮发送，往真实群发了 4 条（2 分钟内撤回）——要么内联那段函数，要么先给脚本补 `if __name__ == "__main__":`。**误触发真的发生了，就先做可撤销动作再查根因**——有撤销窗口的（群消息撤回／还原工作区／回滚提交）一律先执行，窗口比根因分析贵。
- **迭代式脚本开发** — 写主干 → 跑真实数据 → 修问题 → 再跑。适合输入格式不确定的数据处理脚本。见 `references/iterative-script-dev.md`
- **系统化调试** — 找到根因前不下修。四阶段：调查→分析→假设→修复
- **Spike** — 快速验证，不承诺交付。验证→裁决→决定

### 代码质量

- **预提交审查** — 安全扫描→回归基线→自审清单→独立子agent审查
- **代码清理** — 四路子agent并行找复用/质量/效率/深度问题

### SKILL.md 编写

Hermes Agent 技能文件的 frontmatter 和结构规范。详见 `references/skill-authoring.md`。

**索引/目录描述简短化（2026-08-06 纠正）** — 目录、索引、导视是检索表不是百科：条目**一句点题**，细节归叶子文件（各立绘 md/v4 描述/档案正文）。写多了是检索噪音。实例：立绘 INDEX 系列注释从 4 行详情缩成一句「天使系列：羽翼+哥特教堂场景」。自检：这条注释如果删掉一半信息读者还能定位，就是写多了。

## 方法论原则 · 分档速查

「子领域 · 方法论原则」下原先逐条平铺的条目，按场景归了四档（整条搬出，判据与实测一字未改）：

| 你要做的事 | 打开 |
|:--|:--|
| **怎么开工**：长任务先落计划（阶段·产出·验收·卡点）／先摸机制再动产出／方向题先答别先侦查／用户调线就先把那条线做成一轮可交付／逐个来／发现问题即时修 | `references/working-style.md` |
| **要不要做成工具、CLI 怎么改**：一脚本一职责／改自带 CLI 的测试三步／带 INDEX 的 CLI 行解析／做减法不只 py／多步 ad-hoc → 定版 CLI（查证·收档·固化）／加一类机制支持要把同一 CLI 的所有输出口扫一遍／后台哨兵失败必须非零退出 | `references/cli-and-scripting.md` |
| **代码与工具坑**：bash 内联反引号／V4A 多 hunk 锚点／replace 模式补丁／execute_code 断言式 edit／别用 `json.dumps(indent=2)` 重排手写 JSON | `references/cli-authoring-pitfalls.md` |
| **同步·迁移·验收**：状态类工具先沙盒／判「哪边改了」比上游 ref／合并看内容不看 index／推送失败可恢复／对拍自测／反样本证「不乱触发」／对账读内容／机械分类核边缘样本／生成器覆盖前比对零差异／批量改数据连带查断言台账／拆分类改动核「没丢字」／Reference 数值完备／**跨档案批量同步后逐文件回读目标段落**（一次同步里会出现：表头改了表体没跟、插行时漏了同一批的另一行）／**批处理产物核「每一段都在」不核退出码**／共享工作区提交用精确路径、别吞掉别人的未提交增量／**验收工具自己的前置失败必须计入失败与退出码**（「核对不了」不能当通过——假绿会把没核过的改动放进提交） | `references/sync-and-verification.md` |
| **git 推送不通** | `references/git-push-troubleshooting.md` |
| **提交门禁（把检查装到 git 上）** | `references/commit-gates.md` |

## 拆分记录

- **2026-09-25 子领域逐条搬出（SKILL.md 36.5KB → 6.8KB）**：「方法论原则」32 条按场景归四档（working-style／cli-and-scripting／sync-and-verification／并入既有 cli-authoring-pitfalls）；「git 推送排障」「提交门禁」两节整节搬出；SKILL 留 使用哲学／什么时候用／工作流速查／子领域指针 + 本表。搬运逐字，一条未删。
