---
tier: T2  # T分级: T2=直接做 / T1=先请示 / T0=一律拒
name: python-debugpy
description: "Debug Python: pdb REPL + debugpy remote (DAP)."
---

# Python 调试

## 三板斧

| 方式 | 一句话 | 什么时候用 |
|:----|:------|:----------|
| `breakpoint()` + pdb | 源码加一行，跑起来停住 | 本地、最快、最常用 |
| `python -m pdb script.py` | 不用改源码直接跑 | 快速看一眼 |
| `debugpy` / `remote-pdb` | 远程附加到运行中的进程 | 长服务、daemon、子进程 |

## 快速上手

```python
# 在问题位置前加这行
breakpoint()
# 跑起来后会停在 (Pdb) 提示符
# 常用命令：n(下⼀行) s(跳进函数) c(继续) p(打印) w(堆栈)
```


## 参考章节

| 文件 | 内容 |
|:----|:-----|
| `references/01-overview-to-pdb-quick-reference.md` | Overview (52行) |
| `references/02-recipe-1-local-breakpoint-to-recipe-3-debug-a-pytest-test.md` | Recipe 1: Local breakpoint (53行) |
| `references/03-recipe-4-post-mortem-on-any-exception.md` | Recipe 4: Post-mortem on any exception (26行) |
| `references/04-recipe-5-remote-debug-with-debugpy-attac.md` | Recipe 5: Remote debug with debugpy (attach to running process) (129行) |
| `references/05-debugging-hermes-specific-processes-to-verification-checklist.md` | Debugging Hermes-specific Processes (67行) |
| `references/06-one-shot-recipes.md` | One-Shot Recipes (33行) |