---
tier: T1  # T分级: T2=直接做 / T1=先请示 / T0=一律拒
---

# 脚本化减法审计 · 方法 + 2026-08-02 实例

> 场景：被要求「看一眼小工具跟 skill，看看有哪些可以做成脚本来做减法，不只考虑 py 脚本」。
> 这是把「重复劳动 → 脚本」做成系统化审计的方法，不是一次性任务叙事。

## 审计方法（可复用）

1. **扫一遍全部 skill 的 linked_files** — 看哪些高频流水线 skill 已有 `scripts/`，哪些还是纯方法论文档。
2. **三分类**：
   - 已脚本化（bili-archive / douyin / typhoon-monitor / wechat-article / documents / steam-api / taobao-affiliate / location / info-hunt / diary-writer 等）→ 不动
   - 方法论型（news-verification / build-analysis / skill-curation / persona-authoring 等）→ 靠 LLM 推理，脚本化空间小，留文档
   - 高频重复劳动但无脚本 → 候选
3. **候选判断标准**：动作是否**每次重复且确定性**（建模板 / 填固定格式 / 更新索引 / 打包文件）→ 是则可以脚本化；需要 LLM 推理的部分留在 skill 文档。
4. **形式不止 py**：HTML 模板 + 生成器、shell 包装、CLI 工具都算。选形式看动作本质：网页骨架→模板+生成器；命令封装→shell；多步建文件→py CLI。

## 候选清单（2026-08-02 产出，待执行）

| # | 对象 | 现状 | 方案 | 形式 |
|:--|:-----|:-----|:-----|:-----|
| 1 | am-tool-collection（小工具合集，2026-09-26 由 skill 降级为存档目录 `workspace/scripts/am-tool-collection/`） | 每次从零生成整个 HTML | 共享骨架模板 + `am_tool_gen.py <tool-id> [主题]` 产出骨架到 workspace/tools/，LLM 只填核心逻辑 | HTML 模板 + py 生成器 |
| 2 | record-officer-daily | 手动建条目+填5段格式+更新INDEX+提交 | `ro new "标题"` 建模板、`ro done` 提交推送 | py CLI |
| 3 | internet-memes-reference | 加梗=复制模板→填空→手动更新导视表 | `meme add <类目> "梗名"` 一步完成 | py CLI |
| 4 | voice-output | 每次手敲 edge-tts + zip 打包 | `bin/tts "文字"` 默认晓晓，`--zip` 保真档 | shell 包装 |
| 5 | environment-hygiene | 散货靠人眼审计 | `hygiene audit` 扫根目录非白名单 + gitignore 泄漏 | py CLI |
| 6 | 某连载 skill | 新篇=手动建目录+搬图+建 article.md 骨架 | `story new <编号> "标题"` 一步建好 | py CLI |

## 发现的问题

- **某档案库 断链**：SKILL.md 踩坑 #7 写 `python3 <数据根>/tools/strip_html.py`，但 `<数据根>/tools` 目录不存在。**已修复（2026-08-02）**：补写 `<数据根>/tools/strip_html.py`（去 script/style 块 → 块级标签转换行 → 实体解码 → 压缩空行），实测可用。（踩坑现位于 `某档案库/references/06-踩坑记录.md`）

## ⚠️ 新建 skill 的坑：必须用 skill_manage create，不能 write_file 直写

2026-08-02 实测：本会话用 `write_file` 直接写 `skills/link-safety-check/SKILL.md`（外加 scripts/ + bin/ 包装）建了新 skill。功能正常、git 提交了，但**元数据 created_by=None**——后台 curator 拒绝后续自动维护（`Refusing background curator patch ... not agent-created`），连本会话后续想 patch 它补充经验都被拒（link-safety-check / skill-curation / dev-workflow 三个宿主都被拦，dev-workflow 是唯一 read-before-write 要求解除后能写的）。

**正确流程**：`skill_manage(action='create')` 建骨架 → `skill_manage(action='write_file')` 加 references/scripts → `skill_manage(action='patch')` 迭代。
**误建后修复**：改 `.usage.json` 的 `created_by` 字段，或删除后用 create 重建。
**curator 模式特性**：session 内普通 `patch` 工具改 skill 文件是通的；curator 的 `skill_manage` 有独立保护检查（按元数据判 agent-created）+ read-before-write 要求（patch 前必须先 skill_view 加载目标文件）。写新 skill 当天就按 create 流程走，别等 curator 收尸。

## 执行状态（2026-08-02 完成）

全部 6 项候选 + 断链已落地：

| # | 产物 | 命令 | 验证 |
|:--|:-----|:-----|:-----|
| 1 | `workspace/scripts/am-tool-collection/scripts/am_tool_gen.py` + `templates/tool-skeleton.html` | `am_tool_gen.py pattern` | ✅ 生成骨架无残留占位符（2026-09-26 skill 已降级为存档目录，脚本随之搬、实测仍可跑） |
| 2 | `record-officer-daily/scripts/ro.py` + `<数据根>/bin/ro` | `ro new "标题"` / `ro done` | ✅ INDEX 倒序插入正确 |
| 3 | `internet-memes-reference/scripts/meme.py` + `<数据根>/bin/meme` | `meme add 流行梗 "梗名"` | ✅ 建文件+更新导视表 |
| 4 | `<数据根>/bin/tts` | `tts "文字" [--zip]` | ✅ 合成+zip+时长验证 |
| 5 | `environment-hygiene/scripts/hygiene_audit.py` + `<数据根>/bin/hygiene` | `hygiene audit` | ✅ 报告+gitignore检查 |
| 6 | `某连载 skill/scripts/story.py` + `<数据根>/bin/story` | `story new 21 "标题"` | ✅ 目录+article.md+images |
| 7 | `<数据根>/tools/strip_html.py` | `strip_html.py x.html` | ✅ 段落结构保留 |

审计发现待决策散货：根目录 `README.md`（7-31 生成）与 `plans/`（空目录，7-25 建）——按铁律只报告不自动删。
