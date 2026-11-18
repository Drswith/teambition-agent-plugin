# Teambition Agent Plugin

在 Codex、Claude Code 和 WorkBuddy / CodeBuddy 中使用 Teambition 开放平台 MCP。插件包含一个共享技能，用于查询项目和任务、编写 TQL，以及按需查阅评论、附件、工时等操作的服务端文档。客户端通过 OAuth 连接当前用户的 Teambition 账号，数据和操作权限由 Teambition 决定。[Teambition 开放平台文档](https://open.teambition.com/docs/documents/68ad4901f7d70fb6fb33f159)

## 安装

克隆本仓库后，在仓库根目录执行对应客户端的命令。插件 ID 为 `teambition@teambition-plugins`。

### Codex

```sh
codex plugin marketplace add "$PWD"
codex plugin add teambition@teambition-plugins
```

如果客户端显示需要授权，执行以下命令，在浏览器中选择钉钉账号并完成授权。已有同名、同地址的 OAuth 凭据可被复用。

```sh
codex mcp login teambition --oauth-client-registration dcr
```

此前通过 `codex mcp add` 手动接入过 Teambition 的用户，若要由插件管理连接，可以从 Codex 用户配置移除手动添加的同名服务器条目，再在新会话中验证。插件仅声明官方 URL，OAuth 资源地址由客户端发现。显式配置 `oauth_resource` 在 Codex 0.169.0 会重复发送 `resource`，导致 `invalid_target`。

### Claude Code

```sh
claude plugin marketplace add "$PWD"
claude plugin install teambition@teambition-plugins
claude mcp login plugin:teambition:teambition
```

也可以在新会话中通过 `/mcp` 选择插件的 Teambition 服务并发起授权。显式调用技能可用 `/teambition:teambition`。

### WorkBuddy / CodeBuddy

在 WorkBuddy 的「专家·技能·连接器 → 技能 → 套件 → 添加市场」中添加仓库路径，再安装 `teambition` 套件。CodeBuddy CLI 可执行以下命令。

```sh
codebuddy plugin marketplace add "$PWD"
codebuddy plugin install teambition@teambition-plugins
```

在客户端的 MCP 界面查看 Teambition 连接状态，并按提示完成授权。WorkBuddy 桌面客户端的 Teambition OAuth 回调及数据读取仍待验证。[CodeBuddy MCP 文档](https://www.workbuddy.cn/docs/cli/mcp)

### DeepSeek Harness 兼容接入

通过 Teambition 官方 CLI 完成 OAuth 授权，DSH 加载官方 CLI 版 Skill 后在对话中调用该命令。以下终端示例适用于 macOS 和 Linux。[Teambition CLI 与 Skill 指南](https://open.teambition.com/docs/documents/6a561ee8662d4dfb252d0bb8)

1. 用 npm 安装 CLI，在浏览器中使用自己的 Teambition 账号完成授权，再检查登录状态。

   ```sh
   npm install -g @tng/teambition-mcp-cli
   teambition auth login
   teambition auth status
   ```

2. 下载[官方 CLI 版 Skill](https://file.teambition.net/public/teambition-cli/skills/teambition/teambition.zip)，将整个 `teambition/` 目录解压到 DSH 的用户技能目录。默认入口是 `~/.dsh/skills/teambition/SKILL.md`。设置了 `DSH_HOME` 时，使用其下的 `skills/`。如果 DSH 已加载同名技能，确认它使用 `teambition-cli`，并保留压缩包内的 `references/`。[DSH 技能发现路径](https://deepseek-harness.github.io/deepseek-harness/en/reference/subsystems/skills#local-discovery-priority)

3. 重启 DSH，在新会话里输入「用 Teambition Skill 查询我参与的项目，列出项目名称」。也可以先在终端验证同一条只读查询。

   ```sh
   teambition project query --my
   ```

如果 DSH 找不到 `teambition`，确认 DSH 的命令执行环境能访问 npm 的全局命令目录。原生 DSH bundle 计划在 Teambition 业务 MCP 的 OAuth 端到端验证通过后提供。

## 使用

安装和授权后，打开新会话并尝试以下请求。

- 列出我参与的 Teambition 项目。
- 查找跨项目分配给我的未完成任务，按截止时间排序。
- 查一下这个 Teambition 任务的详情和评论。

## 可选接入开放平台文档 MCP

开发 Teambition 集成时，可以接入 `teambition-docs`，用于查找 OpenAPI 参数、鉴权说明、Webhook 事件、错误码和代码示例。查询项目、任务和工时继续使用插件内置的业务 MCP。[开放平台文档检索 MCP](https://open.teambition.com/docs/documents/695b28bcfca732ddb816bd7e)

```sh
# Codex
codex mcp add teambition-docs --url https://open.teambition.com/docs/mcp

# Claude Code
claude mcp add --transport http teambition-docs https://open.teambition.com/docs/mcp

# CodeBuddy
codebuddy mcp add --transport http teambition-docs https://open.teambition.com/docs/mcp
```

使用图形界面时，添加名为 `teambition-docs` 的远程 MCP，传输方式选择 Streamable HTTP，地址填写 `https://open.teambition.com/docs/mcp`。接入后可以问「创建项目任务需要调用哪个 OpenAPI？请列出必填参数并附上文档链接。」

## 插件结构

根目录的 `plugin.json`、`mcp.json` 与 `skills/` 构成 [Agent Plugins 1.0](https://agent-plugins.org/) 标准包。三个客户端共用 `.mcp.json` 和技能目录。官方服务地址同时出现在 `mcp.json` 与 `.mcp.json`，两个文件分别使用标准包和客户端所需的传输类型。

插件图标取自 [Teambition 的 App Store 页面](https://apps.apple.com/cn/app/teambition/id656664814)，采用与 [Teambition 官网](https://www.teambition.com/) 一致的蓝色品牌标志。插件代码采用 MIT 许可。
