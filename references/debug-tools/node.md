---
tier: T2  # T分级: T2=直接做 / T1=先请示 / T0=一律拒
name: node-inspect-debugger
description: "Debug Node.js via --inspect + Chrome DevTools Protocol CLI."
---

# Node.js 调试

## 快速选择

| 工具 | 一句话 | 安装 |
|:----|:------|:-----|
| `node inspect` | 内置 CLI 调试器，零安装 | Node.js 自带 |
| CDP via `chrome-remote-interface` | 可编程调试 | `npm i chrome-remote-interface` |

## 最常用：`node inspect`

```bash
# 启动后停在第一行
node --inspect-brk script.js

# debug> 常用命令
# n(next) s(step) c(cont) bt(backtrace)
# sb('file.js', 42) 设置断点
# repl 进入 REPL（可看/改变量）
```

## 附加到运行中进程

```bash
kill -SIGUSR1 <pid>
node inspect -p <pid>
```


## 参考章节

| 文件 | 内容 |
|:----|:-----|
| `references/01-overview-to-quick-reference-node-inspect-repl.md` | Overview (57行) |
| `references/02-attaching-to-a-running-process.md` | Attaching to a Running Process (31行) |
| `references/03-programmatic-cdp-scripting-from-terminal.md` | Programmatic CDP (scripting from terminal) (73行) |
| `references/04-debugging-hermes-ui-tui-to-running-vitest-tests-under-the-debugger.md` | Debugging Hermes ui-tui (62行) |
| `references/05-heap-snapshots-cpu-profiles-non-interact-to-verification-checklist.md` | Heap Snapshots & CPU Profiles (Non-interactive) (51行) |
| `references/06-one-shot-recipes.md` | One-Shot Recipes (30行) |