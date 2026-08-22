---
kind: external_dependency
name: Local REST API 插件（外部程序访问 Vault）
slug: obsidian-local-rest-api
category: external_dependency
category_hints:
    - vendor_identity
scope:
    - '**'
source_files:
    - .obsidian/community-plugins.json
---

Obsidian 社区插件 obsidian-local-rest-api，暴露本地 HTTP API，允许外部进程（如 Agent、脚本）读取/写入 Vault 中的笔记与元数据。这是实现「用 agent 快速把对话转换成笔记」的关键集成点——外部工具可通过该 REST API 直接操作 .md 文件与 Obsidian 属性。