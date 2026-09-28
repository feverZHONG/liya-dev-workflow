---
tier: T1  # T分级: T2=直接做 / T1=先请示 / T0=一律拒
---

# 环境侦查

> 动手前先摸清环境底牌。

## 什么时候用

- 接到任务但不确认当前环境有什么工具
- skill 依赖的 CLI/Python 包没确认是否可用
- 在新环境/新容器里第一次干活

## 检查清单

```bash
for cmd in node python3 uv git rg go cargo; do
  which "$cmd" && "$cmd" --version 2>&1 | head -1 || echo "❌ $cmd"
done

python3 -c "import pkg; print('ok')" 2>&1 || echo "❌ pkg"

curl -sI --connect-timeout 5 "https://api.example.com" 2>&1 | head -3
```

## ⚠️ 环境变量不随 terminal 继承（2026-08-14 实测）

Hermes `.env` 里的变量（`XIAOMI_API_KEY` / `DEEPSEEK_API_KEY` 等）**不会自动 export 到 terminal 子进程**。Python `os.environ.get()` 拿到空值 ≠ key 不存在。

**正确姿势**：从 `.env` 文件直接读取：
```python
env = {}
with open("<数据根>/.env") as f:
    for line in f:
        if "=" in line and not line.startswith("#"):
            k, v = line.strip().split("=", 1)
            env[k] = v
api_key = env.get("XIAOMI_API_KEY", "")
```

Shell：`grep '^XIAOMI_API_KEY=' <数据根>/.env | cut -d= -f2`

**排查**：`python3 -c "import os; print(os.environ.get('KEY', 'NOT SET'))"` — 输出 `NOT SET` = 继承问题，不是 key 不存在。

## 切换 Provider 后的检查清单（2026-08-14 实测）

换模型/provider 后容易漏的东西：

1. **Cron job 引用** — `cronjob list` 检查所有 job 的 `model_snapshot` / `provider_snapshot`，旧值不会自动跟主 config 变。`cronjob update --model mimo-v2.5 --provider xiaomi` 同步
2. **Fallback 链** — `hermes fallback list` 确认降级链指向可用的备用 provider。备用 key 也要在 `.env` 里
3. **新模型能力验证** — 别假设新模型跟旧模型能力一致。跑一轮核心能力测试（代码/推理/多模态/中文），记下已知限制（如 mimo-v2.5 是推理模型，reasoning 占 30-80%，视觉需 max_tokens≥4000）
4. **config backup** — 换之前 `cp config.yaml config.yaml.bak.$(date +%Y%m%d_%H%M%S)` 保留旧配置
