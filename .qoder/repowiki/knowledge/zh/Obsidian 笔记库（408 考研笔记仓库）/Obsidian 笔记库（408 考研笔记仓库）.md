---
kind: external_dependency
name: Obsidian 笔记库（408 考研笔记仓库）
slug: obsidian
category: external_dependency
category_hints:
    - vendor_identity
scope:
    - '**'
source_files:
    - .obsidian/app.json
    - .obsidian/core-plugins.json
    - .obsidian/community-plugins.json
---

本仓库是一个 Obsidian Vault，所有 Markdown 笔记由 Obsidian 管理。核心配置位于 .obsidian/：app.json 中 attachmentFolderPath 指向本地「附件」目录；core-plugins.json 启用了 file-explorer、graph、backlink、daily-notes、templates、command-palette、sync、bases、webviewer 等官方插件；community-plugins.json 列出了社区插件清单。

该仓库本身不包含 Obsidian 客户端代码，仅作为 Obsidian 的数据源与配置文件集合存在。后续若需通过 Agent 将对话快速转换为笔记，应基于此 Vault 结构（.md 文件 + .obsidian 配置）进行读写操作。