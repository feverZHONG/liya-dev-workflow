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

Hermes `.env` 里的变量（`<厂商>_API_KEY`，如 `DEEPSEEK_API_KEY` 等）**不会自动 export 到 terminal 子进程**。Python `os.environ.get()` 拿到空值 ≠ key 不存在。

**正确姿势**：从 `.env` 文件直接读取：
```python
env = {}
with open("<数据根>/.env") as f:
    for line in f:
        if "=" in line and not line.startswith("#"):
            k, v = line.strip().split("=", 1)
            env[k] = v
api_key = env.get("<厂商>_API_KEY", "")
```

Shell：`grep '^<厂商>_API_KEY=' <数据根>/.env | cut -d= -f2`

**排查**：`python3 -c "import os; print(os.environ.get('KEY', 'NOT SET'))"` — 输出 `NOT SET` = 继承问题，不是 key 不存在。

## 切换 Provider 后的检查清单（2026-08-14 实测）

换模型/provider 后容易漏的东西：

1. **Cron job 引用** — `cronjob list` 检查所有 job 的 `model_snapshot` / `provider_snapshot`，旧值不会自动跟主 config 变。`cronjob update --model mimo-v2.5 --provider xiaomi` 同步
2. **Fallback 链** — `hermes fallback list` 确认降级链指向可用的备用 provider。备用 key 也要在 `.env` 里
3. **新模型能力验证** — 别假设新模型跟旧模型能力一致。跑一轮核心能力测试（代码/推理/多模态/中文），记下已知限制（如 mimo-v2.5 是推理模型，reasoning 占 30-80%，视觉需 max_tokens≥4000）
4. **config backup** — 换之前 `cp config.yaml config.yaml.bak.$(date +%Y%m%d_%H%M%S)` 保留旧配置

## 把外部开源工程拉到本机跑起来（无 GPU 容器的固定套路）

顺序很重要：**先拿到源码 → 再装依赖 → 装浏览器 → 出静帧 → 出小样 → 才谈整片**。

```bash
# ① 源码：直连 clone 断流时走镜像 zip（实测 12MB 级别能下全）
curl -sL -o src.zip "https://ghfast.top/https://github.com/<owner>/<repo>/archive/refs/heads/main.zip"
python3 -c "import zipfile;zipfile.ZipFile('src.zip').extractall('.')"   # 容器常没有 unzip

# ② 依赖：postinstall 会去 GitHub 拉二进制的（ffmpeg-static 一类）直接跳过
npm install --ignore-scripts --registry=https://registry.npmmirror.com --no-audit --no-fund

# ③ 浏览器：playwright 自己的下载器拉不到时，直接下 Chrome for Testing 的 headless shell
#    https://cdn.npmmirror.com/binaries/chrome-for-testing/<version>/linux64/chrome-headless-shell-linux64.zip
#    然后给工程一个 CHROME_PATH / executablePath 入口
ldd <chrome>/chrome-headless-shell | grep -c 'not found'    # 0 才能跑
```

- **系统已有 ffmpeg 就别修 ffmpeg-static**：多数导出脚本是 `which ffmpeg` 优先。先 `ffmpeg -version` 再决定要不要折腾包。
- **`npm install` 中途 kill 会把 `node_modules` 整个回滚掉**——看到卡在 `node install.js` 先换 `--ignore-scripts` 重装，别硬等；等一轮 4 分钟只为拉一个系统本来有的二进制是纯浪费。
- **无 GPU 的 WebGL 走 SwiftShader**：launch 参数要 `--use-angle=swiftshader --enable-unsafe-swiftshader`，Windows 那套 `--use-angle=d3d11` 在 Linux 上直接失败。
- **先测 10 秒样片并读日志的三段耗时**（render / read / send）：常见瓶颈是**读回像素**（每帧 8MB 的 RGBA 回传），不是渲染本身。判据拿到后再决定降分辨率、降帧率还是拆段并行（4 核 ≈ 3 倍）；不先测就上 4K，会把一小时的账算成一天。
- **先静帧再视频**：多数导出器有 `--sheet`／静帧模式，几秒到一分钟就能确认「画面画得对不对」，比渲了十分钟才发现字体缺字便宜得多。
