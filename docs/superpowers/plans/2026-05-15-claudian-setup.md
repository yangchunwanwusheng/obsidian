# Claudian Setup Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 让 `D:\note` 中的 Claudian 显式绑定本机可用的 Claude Code CLI，并验证它能在 Obsidian 中正常工作。

**Architecture:** 本次只修改 Vault 级配置文件 `D:\note\.claudian\claudian-settings.json`，不改全局 Claude 配置，也不改其他 Obsidian 插件设置。实现方式是把 Claudian 的 `providerConfigs.claude.cliPath` 指向 npm 安装包中的真实 `claude.exe`，然后通过文件级校验和 Obsidian 内最小交互测试验证配置生效。

**Tech Stack:** Obsidian, Claudian, Claude Code CLI, JSON 配置文件, Windows

---

## File Map

- Modify: `D:\note\.claudian\claudian-settings.json`
  - 责任：保存当前 Vault 的 Claudian 用户配置；本次仅更新 `providerConfigs.claude.cliPath`
- Verify: `D:\note\docs\superpowers\specs\2026-05-15-claudian-setup-design.md`
  - 责任：记录已批准的配置设计，供实施时对照
- Verify: `D:\note\.obsidian\community-plugins.json`
  - 责任：确认 `claudian` 插件已启用；本次不修改

## Preconditions

- 本机 Claude Code CLI 已安装
- 已确认可执行入口存在：
  - `C:\Users\oobbee\AppData\Roaming\npm\node_modules\@anthropic-ai\claude-code\bin\claude.exe`
- `D:\note` 不是 Git 仓库，因此本计划不包含 commit 步骤

### Task 1: 写入 Claudian CLI 路径

**Files:**
- Modify: `D:\note\.claudian\claudian-settings.json`
- Verify: `D:\note\docs\superpowers\specs\2026-05-15-claudian-setup-design.md`

- [ ] **Step 1: 读取当前配置并确认目标字段**

确认当前 JSON 中存在以下结构，并且 `cliPath` 为空字符串：

```json
{
  "providerConfigs": {
    "claude": {
      "cliPath": ""
    }
  }
}
```

预期：只修改 `providerConfigs.claude.cliPath`，其他字段保持原值。

- [ ] **Step 2: 准备要写入的最小变更**

将目标值设为：

```json
{
  "providerConfigs": {
    "claude": {
      "cliPath": "C:\\Users\\oobbee\\AppData\\Roaming\\npm\\node_modules\\@anthropic-ai\\claude-code\\bin\\claude.exe"
    }
  }
}
```

说明：必须写入 `claude.exe` 的绝对路径，而不是 `claude.cmd` 或 `C:\Users\oobbee\AppData\Roaming\npm\claude`。

- [ ] **Step 3: 修改配置文件**

把 `D:\note\.claudian\claudian-settings.json` 中这一段：

```json
"providerConfigs": {
  "claude": {
    "safeMode": "acceptEdits",
    "cliPath": "",
    "cliPathsByHost": {},
    "loadUserSettings": true,
    "enableChrome": true,
    "enableBangBash": true,
    "enableOpus1M": false,
    "enableSonnet1M": false,
    "lastModel": "opus",
    "environmentVariables": "",
    "environmentHash": ""
  }
}
```

改成：

```json
"providerConfigs": {
  "claude": {
    "safeMode": "acceptEdits",
    "cliPath": "C:\\Users\\oobbee\\AppData\\Roaming\\npm\\node_modules\\@anthropic-ai\\claude-code\\bin\\claude.exe",
    "cliPathsByHost": {},
    "loadUserSettings": true,
    "enableChrome": true,
    "enableBangBash": true,
    "enableOpus1M": false,
    "enableSonnet1M": false,
    "lastModel": "opus",
    "environmentVariables": "",
    "environmentHash": ""
  }
}
```

预期：只有 `cliPath` 一项发生变化。

- [ ] **Step 4: 校验 JSON 和路径是否正确**

运行：

```bash
python - <<'PY'
import json
from pathlib import Path
p = Path(r'D:/note/.claudian/claudian-settings.json')
data = json.loads(p.read_text(encoding='utf-8'))
print(data['providerConfigs']['claude']['cliPath'])
PY
```

Expected output:

```text
C:\Users\oobbee\AppData\Roaming\npm\node_modules\@anthropic-ai\claude-code\bin\claude.exe
```

然后运行：

```bash
ls -la "/c/Users/oobbee/AppData/Roaming/npm/node_modules/@anthropic-ai/claude-code/bin/claude.exe"
```

Expected output: 一条 `claude.exe` 文件记录，说明目标路径真实存在。

### Task 2: 验证 Obsidian 中的 Claudian 可用

**Files:**
- Verify: `D:\note\.claudian\claudian-settings.json`
- Verify: `D:\note\.obsidian\community-plugins.json`

- [ ] **Step 1: 确认插件仍已启用**

确认启用列表中包含：

```json
[
  "claudian"
]
```

说明：如果插件未启用，配置即使正确也不会生效。

- [ ] **Step 2: 让 Obsidian 重新加载 Claudian 配置**

在 Obsidian 中执行以下任一操作：

```text
1. 完全退出并重新打开 Obsidian
2. 或关闭并重新打开 Claudian 侧边栏
3. 或使用 Obsidian 的重载命令（如果你已安装相关命令支持）
```

Expected: Claudian 重新读取 `D:\note\.claudian\claudian-settings.json`

- [ ] **Step 3: 执行最小聊天测试**

在 Claudian 里发送：

```text
你好，请确认你正在 D:\note 中工作，并读取 Vault 顶层结构。
```

Expected: 正常返回响应，不出现以下错误：

```text
spawn claude ENOENT
Claude CLI not found
Failed to start Claude CLI
```

- [ ] **Step 4: 如果仍失败，执行最小补救路径**

如果 Obsidian 中仍无法调用 Claude，则只补充一项环境修复，不同时做多项改动。

优先补救内容：在 `providerConfigs.claude.environmentVariables` 或 Vault 级共享环境变量中加入 Node/npm 相关 PATH，例如：

```text
PATH=C:\Users\oobbee\AppData\Roaming\npm;D:\Program Files\nodejs;%PATH%
```

说明：只有在 Step 3 失败时才执行此补救，避免引入不必要变量。

### Task 3: 交付与记录

**Files:**
- Modify: `D:\note\.claudian\claudian-settings.json`
- Verify: `D:\note\docs\superpowers\specs\2026-05-15-claudian-setup-design.md`
- Verify: `D:\note\docs\superpowers\plans\2026-05-15-claudian-setup.md`

- [ ] **Step 1: 汇总实际变更**

交付时明确说明：

```text
1. 修改了哪个文件
2. 写入了什么 cliPath
3. 是否需要你手动重启 Obsidian
4. 最小测试语句是什么
```

- [ ] **Step 2: 明确当前状态**

按以下两种状态之一给出结果：

```text
A. 已完成配置，等待你在 Obsidian 中点一次验证
B. 已完成配置且已通过你的反馈确认可用
```

- [ ] **Step 3: 给出单一步后续动作**

如果尚未在 Obsidian 中验证，下一步只给这一条：

```text
请现在重启 Obsidian，然后在 Claudian 中发送测试语句。
```

如果已经验证成功，下一步只给这一条：

```text
如需，我可以继续帮你把 Claudian 的 Vault 级工作流再顺手优化一轮。
```
