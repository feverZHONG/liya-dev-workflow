---
tier: T2  # T分级: T2=直接做 / T1=先请示 / T0=一律拒
---

## Debugging Hermes-specific Processes

### Tests
See Recipe 3. Always add `-p no:xdist` or run single tests without xdist.

### `run_agent.py` / CLI — one-shot
Easiest: add `breakpoint()` near the suspect line, then run `hermes` normally. Control returns to your terminal at the pause point.

### `tui_gateway` subprocess (spawned by `hermes --tui`)
The gateway runs as a child of the Node TUI. Options:

**A. Source-edit the gateway:**
```python
# tui_gateway/server.py near the top of serve()
import debugpy
debugpy.listen(("127.0.0.1", 5678))
debugpy.wait_for_client()
```
Start `hermes --tui`. The TUI will appear frozen (its backend is waiting). Attach a client; execution resumes when you `continue`.

**B. Use `remote-pdb` at a specific handler:**
```python
from remote_pdb import set_trace
set_trace(host="127.0.0.1", port=4444)   # in the RPC handler you want to trap
```
Trigger the matching slash command from the TUI, then `nc 127.0.0.1 4444` in another terminal.

### `_SlashWorker` subprocess
Same pattern — `remote-pdb` with `set_trace()` inside the worker's `exec` path. The worker is persistent across slash commands, so the first trigger blocks until you connect; subsequent slash commands pass through normally unless you re-arm.

### Gateway (`gateway/run.py`)
Long-lived. Use `remote-pdb` at a handler, or `debugpy` with `--wait-for-client` if you're restarting the gateway anyway.

## Common Pitfalls

1. **pdb under pytest-xdist silently does nothing.** You won't see the prompt, the test just hangs. Always use `-p no:xdist` or `-n 0`.

2. **`breakpoint()` in CI / non-TTY contexts hangs the process.** Safe locally; never commit it. Add a pre-commit grep as a safety net.

3. **`PYTHONBREAKPOINT=0`** disables all `breakpoint()` calls. Check the env if your breakpoint isn't hitting:
   ```bash
   echo $PYTHONBREAKPOINT
   ```

4. **`debugpy.listen` blocks only if you also call `wait_for_client()`.** Without it, execution continues and your first breakpoint may fire before the client is attached.

5. **Attach to PID fails on hardened kernels.** `ptrace_scope=1` (Ubuntu default) allows only same-user ptrace of child processes. Workaround: `echo 0 > /proc/sys/kernel/yama/ptrace_scope` (needs root) or launch under `debugpy` from the start.

6. **Threads.** `pdb` only debugs the current thread. For multithreaded code, use `debugpy` (thread-aware DAP) or set `threading.settrace()` per thread.

7. **asyncio.** `pdb` works in coroutines but `await` inside pdb requires Python 3.13+ or `await` from `interact` mode on older versions. For 3.11/3.12, use `asyncio.run_coroutine_threadsafe` tricks or `!stmt`-based awaits via `asyncio.ensure_future`.

8. **`scripts/run_tests.sh` strips credentials and sets `HOME=<tmpdir>`.** If your bug depends on user config or real API keys, it won't reproduce under the wrapper. Debug with raw `pytest` first to repro, then re-confirm under the wrapper.

9. **Forking / multiprocessing.** pdb does not follow forks. Each child needs its own `breakpoint()` or `set_trace()`. For Hermes subagents, debug one process at a time.

## Verification Checklist

- [ ] After `pip install debugpy`, confirm: `python -c "import debugpy; print(debugpy.__version__)"`
- [ ] For remote debug, confirm the port is actually listening: `ss -tlnp | grep 5678`
- [ ] First breakpoint actually hits (if it doesn't, you likely have `PYTHONBREAKPOINT=0`, you're under xdist, or execution finished before attach)
- [ ] `where` / `w` shows the expected call stack
- [ ] Post-debug cleanup: no stray `breakpoint()` / `set_trace()` in committed code
  ```bash
  rg -n 'breakpoint\(\)|set_trace\(|debugpy\.listen' --type py
  ```

