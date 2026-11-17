---
tier: T2
---

# 批量把 SKILL.md 压到字数线以下

> 判据（字数为主、行数作辅）与减法哲学在 `skill-curation`（用户自有，勿自动改写）；本档只记批量执行时的动作与坑。

## 什么时候用

- 全库盘点后有一批 SKILL.md 超线（偏肥 >8K 字符、必拆 >12K）。
- 只想按固定套路把一批都压下去，不打算为每个重新设计拆法。

## 套路

1. **先量再选节**：`bin/sk skeleton <skill>` 按字符排序，**只搬最大的一节**通常就够；超标 <1K 搬一节，超标 3K+ 搬两到三节。
2. **整节逐字搬**进 `references/<有意义的名字>.md`，文件头写 `> 本档是 SKILL.md「X」整节原文（拆出，逐字未改）`。
3. **SKILL.md 留速记**：节标题＋节首 1–2 行原文，末尾「其余全文见 `references/<档>`」。**别自己重写摘要**——自写摘要最容易写出原文没有的判据；要压缩就用原文行当速记。
4. **大路由表/索引可以搬**，但正文要留 8–12 行紧凑表＋全表指针；纯检索件全搬走会拆散锚点。
5. **补「拆分记录」一行**：前→后字符数＋守恒结果。
6. 一个 skill 一个提交，推完 `bin/gitpush --status` 归零再下一个。

## 守恒复核（这里最容易假绿）

- **别拿原节当基准**：`[l for l in 原节 if l not in 原节]` 恒为空，报「0 缺失」是假的。
- 正确做法：`git show HEAD:skills/<skill>/SKILL.md` 取拆前文本 → 定位同一节 → 检查每条 >40 字长行是否出现在**新 ref 文件**里 → 0 缺失才算过。
- 拆完文件体积要合常识：搬了一大节还几乎没降，就是节边界判错（代码围栏里的 `#`、`###` 子节被父节吞掉）。

## 资料库型 skill 的特殊路径

数据层已归 `workspace/kb/<slug>/` 的 skill，**`skills/<name>/references/` 可能不存在**，直接写会 `FileNotFoundError`。动手前 `ls` 一眼：在 kb 的就写 `workspace/kb/<slug>/references/`，提交时把 kb 与 skill 一起 `git add`。

## 收尾

- 拆到 7.8–8.0K 的记进「贴线名单」，下次加内容先清一节。
- 全库复查：`bin/sk stats --sort chars`，确认超线数为 0，并记下总字符降幅。
- 推不上去时按 `references/git-push-troubleshooting.md` 走（HTTPS+token 优先，API 兜底）。
