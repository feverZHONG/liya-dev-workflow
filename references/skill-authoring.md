---
tier: T1  # T分级: T2=直接做 / T1=先请示 / T0=一律拒
---

# Skill Authoring · 编写 SKILL.md

> 为 Hermes Agent 编写技能文件。

## SKILL.md 结构

```yaml
---
name: skill-name
description: "一句话说明这个 skill 干什么"
---
```

## 核心要求

- 有 YAML frontmatter（name/description 必填）
- description 控制在 100 字内
- 内容聚焦：告诉 agent 什么时候用 + 怎么用
- 不要写大段背景介绍
