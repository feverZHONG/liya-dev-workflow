---
tier: T1  # T分级: T2=直接做 / T1=先请示 / T0=一律拒
---

# 脚本膨胀 → 三模块拆分模式

> 单文件脚本长到 500+ 行时的减法方案。2026-08-02 实测：link_check.py 525 行 → 三模块（367+117+86），回归全过。

## 触发信号

- 脚本 >400 行，且**数据定义和判定逻辑混在一起**
- 每轮实战/每次加规则都要在长文件里找位置改（白名单、关键词、阈值）
- 有「可选增强」功能（调用外部 API）混在主流程里，没 key 的环境也要 import 它

## 三模块分工

| 模块 | 职责 | 维护场景 |
|:-----|:-----|:---------|
| `xxx_rules.py` | **纯数据**：白名单/黑名单/关键词/阈值/常量 | 加规则只碰它，顶部一眼可见 |
| `xxx_enhance.py` | **可选增强**：外部 API 封装（有 key 才调用） | 改情报源只碰它；没 key 环境不 import 成本 |
| `xxx.py`（主） | **判定逻辑 + CLI** | 改逻辑只碰它 |

## 判定拆分的依据

1. **数据 vs 逻辑**：字典/集合/列表常量（TLD 名单、品牌表、白名单映射、关键词、评分阈值）→ 全部进 rules 文件
2. **可选 vs 核心**：外部 API 调用（微步威胁情报这类，无 key 静默跳过）→ 独立文件；核心判定逻辑依赖它时用 `import xxx_enhance` 并让增强函数内部处理「无 key 返回空」
3. **入口只留编排**：主文件保留 analyze() 编排 + render + CLI，函数体变短

## 实测收益（link-safety-check 案例）

- link_check.py：525 → 367 行（-201 行数据/增强挪走）
- 新增 link_rules.py（117 行纯数据）+ threatbook.py（86 行增强）
- 加白名单从「在 500 行逻辑里找位置」变成「rules 文件顶部加一行」
- 没 key 的环境不用 import requests 链

## 第二轮拆分：检查器函数化 + 批量工具（2026-08-02）

单文件 367 行 → 再拆成**检查器模块**（每个判定维度一个函数）+ **批量工具**：

| 模块 | 职责 | 行数 |
|:-----|:-----|:-----|
| `link_checks.py` | 13 个检查器，每个判定维度一个独立函数 | 238 |
| `batch_check.py` | 批量扫描（并发 8，之前临时 shell 循环的固化） | 87 |
| `link_check.py`（主） | 纯编排：analyze 按序调检查器 + render + CLI | 233 |

检查器签名统一 `check_xxx(ctx, findings, score) -> (findings, score)`，ctx 是预解析上下文 dict——**跨步骤状态（is_official_brand/is_trusted_shortener/redirect_ok）通过 ctx 传递**，而不是挤在单个大函数里。

## ⚠️ 拆分后回归会逼出隐藏 bug（真实案例）

**拆分不是纯重构——它是暴露隐藏问题的机会。** link-safety-check 拆分后跑全场景回归，逼出两个此前单函数里没暴露的真实 bug：

1. **生态主域必须进品牌列表**：`snsyun.baidu.com` 被误判「随机短域名」——根因是 `baidu` 不在 `BRANDS`，`brand_lookup` 返回 None，走不到品牌豁免分支。修法：baidu/modian/mihoyo/pizzahut/didi/kuaishou 等生态主域补进 `BRANDS`+`BRAND_DOMAINS`，子域才被正确识别为官方品牌。
2. **启发式规则要用「本质特征」判定，不是「集合枚举」**：随机域启发式首版「注册域 ≤6 字符即可疑」→ 批量扫描 20 个 URL 误报 11 个（github.com/csdn.net 都被标红）。改判定本质：**短串必须含数字才算可疑**（1rk/xsx700/k7z 全是数字+字母混合，github/csdn 纯字母是合法品牌名）。误报从 11 降到 3（剩余是 17utt/uc129 这类真·数字域名，警示合理）。
3. **批量扫描工具首跑就是试金石**：batch_check.py 一跑就暴露误报——不要只测单条输入，批量喂真实数据是启发式规则的验收方式。

**教训**：拆分后「全场景回归」不是可选项——把历史判定过的输入（钓鱼/官方/白名单/随机域）全部重跑，评分分级必须一致；不一致的差异点就是隐藏 bug 或设计缺陷，当场修。

## 注意事项

- 拆分后**必须全场景回归**——把历史判定过的输入全部重跑一遍，评分/分级保持一致（本案例 10 个场景全过）
- 模块间用 `from xxx_rules import (...)` 显式导入，不 `import *`——数据文件改名/删条目时主文件立刻暴露
- 可选增强模块内部自处理降级（`if not HAS_REQUESTS or not key: return {}`），主流程不用 try/except 包它
- 阈值（SCORE_HIGH/SCORE_MED）也进 rules——评分规则变了只改一处

## 变体二：CLI 多子命令脚本 → 薄壳入口 + 模块包（2026-08-29 ww.py 实测）

900 行单体 CLI（argparse 十多个子命令：API 查询 + 外部源 + 本地档案操作 + 工作流混一起）不适合三模块（数据/逻辑切不开，全是函数）。改用**按职责拆包 + 薄壳入口**：

```
scripts/ww.py          # 薄壳：argparse 注册 + 分发给 wwlib.*，无业务逻辑
scripts/wwlib/         # 包：按职责拆 5 模块
├── kurobbs.py         # API 层（网络/格式化）
├── sources.py         # 外部源（百度/萌娘）
├── archive.py         # 本地档案操作（骨架/lint/INDEX）
├── story.py           # 剧情拆档
└── workflow.py        # 多步工作流（build/audit/profile）
```

| 坑 | 现象 | 修法 |
|:---|:-----|:-----|
| **软链入口 sys.path** | `<数据根>/bin/ww` 是软链 → scripts/ww.py；运行 `ww` 时 `sys.path[0]` 是 **bin/ 不是 scripts/**，`import wwlib` 直接 ModuleNotFoundError | 入口顶部、`import wwlib` **之前**：`sys.path.insert(0, os.path.dirname(os.path.realpath(__file__)))`。用 `realpath` 不是 `__file__`，否则拿到的还是软链目录 |
| **模块依赖方向** | 循环 import（workflow 要调 story+archive+kurobbs）| 单向分层：API 层 ← sources/archive ← story/workflow。新功能按层放对应模块，不跨层倒引用 |
| **注册函数与 lint 联动** | 拆分后顺手发现：`register_index` 追加 INDEX 行不带 `.md` 链接 → 生成骨架后 lint 必报「未注册」 | 追加行带 `[name.md](name.md)` 链接，lint 按文件名匹配才能过。**生成器/注册器与校验器要闭环验证**——new 一个骨架 → lint → 删除，一条命令测穿 |

**回归测试也要覆盖「生成→校验」闭环**：`ww new 测试角色` → `ww lint characters/测试角色.md` → 清理。比只测查询命令更能暴露注册类隐藏 bug。

## 变体三：多 CLI 按需求域拆入口 + INDEX 分节（2026-08-29 用户指示「整多个cli来应对多个需求」）

包拆完仍是一个 CLI 塞十几条子命令时，**按需求域拆多个 CLI 入口**（查询/建档维护/校验各归各），bin 各挂一条软链。同源同包，只是入口文件分开。

```
scripts/ww.py        # 查询 CLI：kurobbs/baike/moegirl/profile（bin/ww）
scripts/ww_admin.py  # 建档 CLI：new/harvest/links/lint/audit/build（bin/ww-admin）
scripts/wwlib/       # 共享实现包，两个入口都 import
```

| 坑 | 现象 | 修法 |
|:---|:-----|:-----|
| **新入口没执行权限** | 建完软链跑 `ww-admin` 报 Permission denied | `chmod +x scripts/ww_admin.py`（write_file 创建默认不可执行）|
| **旧命令引用散落** | 命令挪了入口后，文档/SKILL/脚本 print 提示里全是旧路径（`ww build` → `ww-admin build`）| 拆完**全局搜旧命令字符串**（含 `ww kurobbs harvest`/`ww lint` 等子命令），文档+脚本提示一起改，别只改入口文件 |

**归档型 skill 的 INDEX 按类分节**（story/ 按类型、characters/ 按势力分 `## 小节`），目的：打开 INDEX 一眼确认「哪类有什么、缺什么」：

- **自动追加必须归节**：`harvest` 这类自动登记脚本若只是 `content += 一行` 追加到文件末尾，会破坏分节结构。改成按类型找对应 `## 小节` 定位插入（find 小节起始 → find 下一个小节 → 插在末尾前）。
- **类型推断规则要匹配实际文件名后缀**：`("-事件", "活动事件档")` 匹配不上「黎乔利岛事件」（结尾是「岛事件」不是「-事件」）→ 推断返回「待填」、新建错误小节。规则写实际结尾：`("事件", ...)` 不带 `-` 前缀。**改完必须用真实文件名断言测试**（infer_story_kind("老人鱼海-黎乔利岛事件.md") == "活动事件档"）。
- **lint 的「未注册」检查按文件名匹配**：INDEX 分节后 lint 仍能工作（只要文件名出现在 content 里），但 register_index 追加行要带 `[name.md](name.md)` 链接才能过。
