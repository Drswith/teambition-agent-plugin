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

## 使用

安装和授权后，打开新会话并尝试以下请求。

- 列出我参与的 Teambition 项目。
- 查找跨项目分配给我的未完成任务，按截止时间排序。
- 查一下这个 Teambition 任务的详情和评论。

## 插件结构

根目录的 `plugin.json`、`mcp.json` 与 `skills/` 构成 [Agent Plugins 1.0](https://agent-plugins.org/) 标准包。三个客户端共用 `.mcp.json` 和技能目录。官方服务地址同时出现在 `mcp.json` 与 `.mcp.json`，两个文件分别使用标准包和客户端所需的传输类型。

插件图标取自 [Teambition 的 App Store 页面](https://apps.apple.com/cn/app/teambition/id656664814)，采用与 [Teambition 官网](https://www.teambition.com/) 一致的蓝色品牌标志。插件代码采用 MIT 许可。
