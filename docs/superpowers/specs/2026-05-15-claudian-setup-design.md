# Claudian in Obsidian 配置设计

## 目标

将 `D:\note` 这个 Obsidian Vault 中已安装的 Claudian 插件配置到“可用 + 顺手优化”状态，使其能够稳定调用本机已安装的 Claude Code CLI。

## 当前状态

- Vault 路径：`D:\note`
- Claudian 已安装：`D:\note\.obsidian\plugins\claudian`
- Claudian 已启用：`D:\note\.obsidian\community-plugins.json`
- Claudian Vault 配置已存在：`D:\note\.claudian\claudian-settings.json`
- 当前 Claude provider 已启用，但 `providerConfigs.claude.cliPath` 为空
- 本机 `claude` 可运行，但为 npm 全局安装
- 实际 Claude Code 可执行入口已确认存在：
  - `C:\Users\oobbee\AppData\Roaming\npm\node_modules\@anthropic-ai\claude-code\bin\claude.exe`

## 已评估方案

### 方案 A：依赖自动发现
不配置 `cliPath`，继续让 Claudian 自动寻找 Claude CLI。

- 优点：零修改
- 缺点：在当前 Windows + npm 全局安装场景下，自动发现更容易落到 `claude.cmd` 包装器，不稳定

### 方案 B：显式绑定 Claude CLI 路径（推荐）
在 Vault 级 Claudian 配置中，显式指定 Claude Code 的真实可执行路径。

- 优点：最小改动、作用域只在当前 Vault、稳定性最好
- 缺点：需要维护一个本机路径

### 方案 C：改为原生安装后再配置
卸载或绕开 npm 版本，改用原生 Windows 安装的 Claude CLI，再让 Claudian 指向原生路径。

- 优点：长期最稳
- 缺点：侵入性更强，不符合本次“就地配置”的目标

## 选定方案

采用 **方案 B：显式绑定 Claude CLI 路径**。

## 计划修改

只修改以下 Vault 级配置文件：

- `D:\note\.claudian\claudian-settings.json`

### 修改项

将：

- `providerConfigs.claude.cliPath`

设置为：

- `C:\\Users\\oobbee\\AppData\\Roaming\\npm\\node_modules\\@anthropic-ai\\claude-code\\bin\\claude.exe`

### 保留项

以下现有设置保持不变：

- `permissionMode: "yolo"`
- `locale: "zh-CN"`
- `providerConfigs.claude.safeMode: "acceptEdits"`
- `providerConfigs.claude.enableBangBash: true`
- 当前模型与 effort/thinking 相关设置

### 暂不修改项

本次先不主动修改以下内容，避免引入不必要变量：

- `sharedEnvironmentVariables`
- `providerConfigs.claude.environmentVariables`
- Obsidian 其他插件配置
- 全局 Claude Code 配置
- 系统 PATH

如果后续验证时发现 Obsidian 进程内仍无法启动 Claude，再追加环境变量修复。

## 验证方案

### 静态验证

1. 确认 `claudian-settings.json` 为合法 JSON
2. 确认 `cliPath` 已写入且路径与实际文件存在一致

### 运行验证

1. 在 Obsidian 中重载 Claudian 或重启 Obsidian
2. 打开 Claudian 聊天侧边栏
3. 发起一个最小测试请求，例如：
   - `你好，请确认你正在 D:\note 中工作`
4. 成功标准：
   - 不出现 CLI not found / spawn ENOENT 类错误
   - 能正常返回响应
   - 工作目录表现为当前 Vault

## 风险与边界

### 风险

- 路径是机器本地绝对路径，如果未来 Claude Code 安装位置变化，需要同步更新
- 当前 Vault 不是 Git 仓库，无法通过 Git commit 保存本次设计或修改历史

### 边界

- 仅处理 `D:\note` 的 Claudian 可用性与轻量优化
- 不扩展到完整 Obsidian 工作流改造
- 不变更其他无关插件和目录结构

## 完成定义

完成的标准是：

1. `cliPath` 已正确配置
2. Claudian 能在 Obsidian 中成功调用 Claude Code
3. 没有引入额外配置副作用
4. 用户获得最小可用测试方法
