---
tier: T1  # T分级: T2=直接做 / T1=先请示 / T0=一律拒
---

# 系统化调试

> 找到根因之前不下修。症状修复就是失败。

## 什么时候用

任何技术问题——测试失败、生产 bug、构建报错、集成问题。越是紧急越该用。

## 四阶段

| 阶段 | 一句话 | 详细 |
|:----|:-------|:-----|
| ① 根因调查 | 读错误→建复现命令→查最近变更→追数据流 | 先建反馈回路 |
| ② 模式分析 | 极小化复现→找工作案例→对比差异 | 缩小范围 |
| ③ 假设验证 | 列出 3-5 个可证伪假设→排优先级→测一个 | 每次只改一个变量 |
| ④ 实施修复 | 先写失败测试→单次修改→验证→全量跑 | 确认根因已被消灭 |

## 铁律

```
找到根因之前不下修。没找到根因前不碰代码。
```

## 工具手册（按语言查）

| 语言 | 快速上手 | 详细 |
|:----|:---------|:-----|
| Python | `breakpoint()` → (Pdb) n/s/c/p | `references/debug-tools/python.md` |
| Node.js | `node --inspect-brk script.js` → `node inspect` | `references/debug-tools/node.md` |

> 2026-09-28 从独立 skill `debug` 并入（原触发位撤销）——本档管**方法论**（怎么想），下面这 14 份管**工具**（怎么敲）。
> `references/debug-tools/` 全部清单：
> - Python：pdb 速查｜本地断点→pytest 三个配方｜异常后验尸｜debugpy 远程附加｜Hermes 进程调试与验证清单｜一次性配方
> - Node：inspect REPL 速查｜附加运行中进程｜CDP 脚本化｜Hermes UI-TUI 与 vitest 调试｜heap snapshot 与 CPU profile｜一次性配方
