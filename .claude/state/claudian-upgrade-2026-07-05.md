# Claudian upgrade state

Date: 2026-07-05
Vault: `D:\note`

## 改动一览

| 项 | 改前 | 改后 |
|---|---|---|
| 插件版本 | 2.0.1 (2026-04-08) | **2.0.27** (2026-06-29) |
| 插件 ID | `claudian` | `realclaudian`(上游已统一改 ID) |
| 插件目录 | `.obsidian/plugins/claudian/` | `.obsidian/plugins/realclaudian/` |
| `community-plugins.json` | `["claudian", ...]` | `["realclaudian", ...]` |
| `excludedTags` 拼写 | `["privete"]` | `["private"]`(已修) |
| `.claude/commands/` | 空 | + `ingest.md` `daily.md` `lint-wiki.md` |
| `.claude/skills/` | 空 | + `paper-note/SKILL.md` `kb-init/SKILL.md` |
| `.claude/agents/` | 空 | + `wiki-curator.md` `literature-reviewer.md` |

## 为什么改

1. **升级**:claudian 上游已发布 26 个版本,本次升级到 2.0.27 拿到 Codex/Opencode/Pi 多 provider、`@mention` 子代理、`#` 指令模式、MCP、inline edit diff 等能力
2. **ID 改名**:上游把 plugin ID 从 `claudian` 改为 `realclaudian`(社区规范要求 ID 必须唯一;旧 ID 在 BRAT 等安装器中有冲突)
3. **修拼写**:`excludedTags: ["privete"]` 一直是无效值,改成 `["private"]` 后才能正确排除 private 标签的笔记
4. **补内容**:`.claude/{commands,skills,agents}` 三个目录此前为空,补 vault 级示例以便 Claudian 直接识别

## 已验证

- 配置文件 `claudian-settings.json` 关键字段全部解析正常(`userName`/`permissionMode`/`model`/`excludedTags`/`locale`/`cliPath`/`enableOpus1M`)
- 新插件主目录三个文件大小与 GitHub release sha256 一致:
  - `main.js` 4,267,575 B
  - `manifest.json` 447 B
  - `styles.css` 138,454 B
- Claude Code CLI 在配置路径仍可执行:`2.1.201 (Claude Code)`
- 新 `data.json` 已迁移(tabManagerState 三个未关闭的标签页保留)
- `community-plugins.json` 已替换 ID,只启用 `realclaudian`

## 未在 CLI 中验证(需要 Obsidian 内手动验证)

1. 重启 Obsidian,确认无 `Failed to enable plugin: Plugin "realclaudian" requires newer Obsidian version`(minAppVersion 1.7.2)
2. 打开 Claudian 侧边栏,确认:
   - 没有 `spawn claude ENOENT` / `Claude CLI not found`
   - 模型显示 `opus[1m]`,provider 显示 Claude
3. 在 Claudian 中发:"请确认你在 D:\note 工作,读取 wiki/index.md 并列出最近 3 条 log"
4. 测试 `/ingest raw/lessons/...` 是否出现在 Claudian 命令面板(应该出现在 `commands/` 文件夹的内容里)
5. 测试 `@wiki-curator` 与 `@literature-reviewer` 提及,看 agents 是否加载
6. 确认 3 个历史标签页(conv-1780403876667 / conv-1778903512160 / conv-1780464104172)能否正常打开

## 备份与回滚

- 旧插件目录备份:`D:\note\.obsidian\plugins\claudian.bak-20260705-174426\`
- settings 临时备份:`%LOCALAPPDATA%\Temp\claudian-upgrade-backup\`
- 如果新版有致命 bug,回滚步骤:
  1. 退出 Obsidian
  2. 把 `.bak-20260705-174426/` 复制回 `claudian/`
  3. `community-plugins.json` 改回 `["claudian", ...]`
  4. 删 `realclaudian/` 目录
  5. 重启 Obsidian

## 待用户处理(非阻塞)

- [ ] 验证 Obsidian 中 Claudian 2.0.27 正常工作后,删除 `.bak-20260705-174426/` 备份
- [ ] 旧 `claudian/` 目录目前未被 community-plugins.json 引用,但仍在 `.obsidian/plugins/`;Obsidian 会视为已禁用的旧插件,不加载也不会冲突。确认新版无问题后一并删除
- [ ] `userName` 字段仍为空(根 `CLAUDE.md` 未要求,保持空)
- [ ] `.obsidian/mcp.json`(若需要 MCP):当前 `.claude/` 内无此文件,2.0.27 起 Claudian 自管 MCP,需要在 Claudian 设置里添加
