# Obsidian 集成资产检测

## 一、Qoder 侧

Qoder 本机无 Obsidian 专属 MCP 与 skill，但具备 vault 文件直接读写能力，作为笔记写入主通道。

## 二、Vault 侧（d:\笔记）

| 插件 | 作用 |
|---|---|
| `obsidian-local-rest-api` | HTTP API（HTTPS 27124），可供外部工具操作笔记 |
| `realclaudian` | Obsidian 内运行 Claude 会话 |
| `obsidian-git` | 版本管理与同步 |

其他已启用：`templater-obsidian`、`dataview`、`obsidian-kanban`、`tag-wrangler`、`copilot`、`obsidian-excalidraw-plugin`、`terminal` 等；另有 `folder-notes` 已安装但未启用。

## 三、Trae CN 可移植资产

- 3 个 skill：`obsidian-cli`、`obsidian-markdown`、`obsidian-bases`（已移植至本目录 `skills\`）
- MCP 配置：`npx -y obsidian-mcp "D:/笔记"`（文件系统直操作，无需 Obsidian 运行）
- 本机 Obsidian CLI：`D:\Obsidian\Obsidian.com`（备用通道）

> **考点提示**：本文为 agent 冒烟测试产物，确认集成链路正常后可保留作资产清单或删除。
