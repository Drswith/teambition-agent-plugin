---
name: teambition
description: 使用 Teambition 开放平台 MCP 查询和管理项目、任务、迭代、评论、附件和工时。用户提到 Teambition 链接、我参与的项目、分配给我的任务、TQL 或要求更新 Teambition 数据时使用。
---

# Teambition

使用当前客户端暴露的 Teambition 开放平台 MCP 工具。下文使用服务端工具名，客户端可能添加命名空间或转换分隔符。遇到未登录或权限不足时，报告服务端给出的原因及所需授权。

## 查询项目和任务

- 查询我参与的项目，用 `teambition.project.query` 传 `my=true`。已有项目 ID 时传 `projectId`。按名称查项目用 TQL，遇到同名项目时先确定目标。
- 已有任务 ID 或父任务 ID，用 `teambition.task.query` 传 `taskId` 或 `parentTaskId`。查指定项目内分配给我的未完成任务，用 `teambition.project.task-query` 传 `projectId`、`assignedToMe=true` 和 `unfinished=true`。
- 需要编写或修正 TQL 时，先用 `teambition.docs.get` 读取 `task/tql`，同一会话可复用。跨项目任务用 `teambition.task.query` 的 TQL 模式，明确项目和人员范围。跨项目查询分配给我的未完成任务需要同时包含 `executorId = me()` 和 `isDone = false`。修正 TQL 时保留原有范围、人员、状态、日期及排序。
- 需要齐全结果时，沿用原筛选和排序，将响应的 `nextPageToken` 原样传给 `pageToken`，直到游标为空。去重时使用项目 `id` 或任务 `taskId`。

## 执行其他操作

- 不熟悉的能力或多步操作，先用 `teambition.docs.search` 定位文档，再用 `teambition.docs.get` 获取用法。工具参数以当前会话中的 MCP 定义为准。
- 用户明确要求写入后，先确定项目或任务 ID，并核对涉及的成员、状态、任务类型等值。工具要求确认时，按其返回的流程确认。写入超时或结果未知时，先查询结果再决定是否重试，避免重复创建、评论或报工时。
- 上传附件按内置工作流申请上传凭证，通过客户端将文件字节上传至返回的 URL，再用对应工具关联上传结果。远程 MCP 无法读取本地文件路径。任务附件文档为 `workflows/upload-file-to-task`，项目文件文档为 `workflows/upload-project-file`。
