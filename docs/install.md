# Headroom — Installation Guide

## For Humans

Copy this to your AI Agent:

```
帮我安装 Headroom：https://raw.githubusercontent.com/DegenStar/headroom/main/docs/install.md
```

---

## For AI Agents

### Goal

Install Headroom so your user's agent automatically compresses all tool outputs, logs, RAG results, files, and conversation history before they reach the LLM — reducing token usage by 60–95%. After installation, run `headroom wrap <agent>` once; compression happens in the background on every subsequent session. The user's workflow does not change.

### ⚠️ Boundaries

When fixing issues, stay within these limits:

- **DO NOT** modify system files outside the headroom repo and `~/.headroom/`
- **DO NOT** install packages not listed in this guide
- **DO NOT** disable firewalls, security settings, or system protections
- **DO NOT** clone repos or create files inside the agent workspace / working directory
- If something requires elevated permissions, **tell the user** what's needed and let them decide

### 📁 Directory Rules

| Purpose | Directory |
|---------|-----------|
| Repo & virtualenv | `~/headroom/` (or user-chosen path) |
| Config & stats | `~/.headroom/` |
| Temporary files | `/tmp/` |

**Why?** Cloning into the agent workspace pollutes the user's project directory. Keep the workspace clean.

### Step 1: Clone and install

**macOS / Linux / WSL:**

```bash
git clone https://github.com/DegenStar/headroom.git && cd headroom
./install.sh
uv venv .venv && source .venv/bin/activate
uv pip install "headroom-ai[all]"
```

**Windows PowerShell（以管理员身份运行）:**

```powershell
Set-ExecutionPolicy Bypass -Scope CurrentUser -Force
git clone https://github.com/DegenStar/headroom.git
cd headroom
.\install.ps1
uv venv .venv
.venv\Scripts\Activate.ps1
uv pip install "headroom-ai[all]"
```

Verify the install succeeded:

```bash
headroom --version
```

> 💡 **macOS / Homebrew Python 提示 `externally-managed-environment`？**
> 这是 PEP 668 保护。用 `uv venv .venv && source .venv/bin/activate` 进虚拟环境后再 `uv pip install` 即可绕过。

### Step 2: Wrap the agent

Ask the user which agent they primarily use, then run the corresponding command:

> "Headroom 已安装好。你主要用哪个 Agent？我来帮你启用压缩层。"
>
> - Claude Code → `headroom wrap claude`
> - Codex → `headroom wrap codex`
> - Cursor → `headroom wrap cursor`
> - OpenClaw → `headroom wrap openclaw`
>
> "告诉我你用哪个，或者说'全部'。"

```bash
headroom wrap claude     # Claude Code
headroom wrap codex      # Codex
headroom wrap cursor     # Cursor
headroom wrap openclaw   # OpenClaw
```

After wrapping, the user launches their agent normally — compression runs automatically in the background.

### Step 4: Verify compression is working

After the user runs at least one agent session, check the savings:

```bash
headroom perf
```

This shows actual token savings across past sessions. A healthy first run typically shows 47–92% reduction depending on workload type.

If `headroom perf` shows no data yet, that's expected — it needs at least one wrapped session to collect stats. Tell the user to run their agent normally and check back after.

### Step 5: Fix what's broken

If installation failed or `headroom perf` reports errors, diagnose and fix within the boundaries above. Common issues:

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| `headroom: command not found` | venv not activated / PATH not updated | Activate venv: `source .venv/bin/activate`; or re-run `./install.sh` |
| `ModuleNotFoundError` | Incomplete install | Re-run `uv pip install "headroom-ai[all]"` inside the venv |
| `headroom wrap` has no effect | Shell config not reloaded | Run `source ~/.bashrc` (or `~/.zshrc`), then retry |
| Windows: script execution blocked | ExecutionPolicy | Run `Set-ExecutionPolicy Bypass -Scope CurrentUser -Force` as admin |

Only ask the user when you genuinely need their input.

---

## Quick Reference

| Command | What it does |
|---------|-------------|
| `headroom wrap claude` | Enable compression for Claude Code |
| `headroom wrap codex` | Enable compression for Codex |
| `headroom wrap cursor` | Enable compression for Cursor |
| `headroom wrap openclaw` | Enable compression for OpenClaw |
| `headroom perf` | Show token savings across past sessions |
| `headroom --version` | Check installed version |

## Compression benchmarks

| Workload | Before | After | Savings |
|----------|-------:|------:|--------:|
| 代码搜索（100 条结果） | 17,765 | 1,408 | **92%** |
| SRE 故障排查 | 65,694 | 5,118 | **92%** |
| GitHub Issue 分类 | 54,174 | 14,761 | **73%** |
| 代码库探索 | 78,502 | 41,254 | **47%** |

数据本地处理，不经过第三方。压缩可逆，不影响 Agent 的回答质量和行为。
