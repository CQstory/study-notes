---
kind: configuration_system
name: Obsidian 知识库配置体系
category: configuration_system
scope:
    - '**'
source_files:
    - .obsidian/app.json
    - .obsidian/appearance.json
    - .obsidian/core-plugins.json
    - .obsidian/community-plugins.json
    - .obsidian/workspace.json
    - .obsidian/templates.json
    - .obsidian/graph.json
---

## 1. 系统/方法概述

本仓库并非传统意义上的应用程序，而是一个基于 **Obsidian** 的个人考研复习知识库。其“配置系统”完全由 Obsidian 在仓库根目录下的 `.obsidian/` 隐藏文件夹中的 JSON 配置文件构成，用于管理编辑器行为、插件、工作区布局、模板路径等。

没有自定义的 `config/`、`.env`、`*.yaml`、`application.properties` 等应用级配置；所有运行时偏好均以 Obsidian 原生配置文件的键值对形式声明。

## 2. 关键文件与位置

- `.obsidian/app.json`：Obsidian 全局应用设置（如 `alwaysUpdateLinks`、`attachmentFolderPath`）。
- `.obsidian/appearance.json`：外观主题相关设置（当前为空对象 `{}`，表示使用默认外观）。
- `.obsidian/core-plugins.json`：启用/禁用 Obsidian 内置核心插件的开关列表（如 file-explorer、graph、backlink、templates、sync、bases 等）。
- `.obsidian/community-plugins.json`：已安装并启用的第三方社区插件清单（dataview、templater-obsidian、obsidian-git、copilot、realclaudian、terminal 等）。
- `.obsidian/workspace.json`：完整的工作区状态快照，包含左右面板布局、打开的标签页、搜索查询、最近打开文件列表 (`lastOpenFiles`) 等。
- `.obsidian/templates.json`：指定模板存放目录为 `模版`。
- `.obsidian/graph.json`、`.obsidian/plugins/*`：图谱渲染与插件自身配置（未展开阅读）。

## 3. 架构与约定

### 3.1 配置分层
Obsidian 的配置按职责自然分层：
- **应用层** (`app.json`)：控制链接更新策略、附件存储路径等全局行为。
- **界面层** (`appearance.json`)：主题与视觉偏好。
- **功能开关层** (`core-plugins.json` + `community-plugins.json`)：以布尔值或字符串数组声明启用的插件，形成“功能模块”级别的配置。
- **会话层** (`workspace.json`)：每次关闭 Obsidian 时持久化的工作区状态，属于易变配置，不应手动编辑。
- **模板层** (`templates.json`)：将模板集中到 `模版/` 目录，配合 Templater / Templates 插件实现笔记自动生成。

### 3.2 内容组织约定
- 附件统一存放在根目录 `附件/`，由 `app.json` 中 `attachmentFolderPath: "附件"` 强制约束。
- 笔记按学科分目录：`408/`（计算机考研四科）、`数一/`（数学一），顶层散落少量通用公式笔记。
- 模板集中放在 `模版/` 目录，由 `templates.json` 指向。
- 知识单元之间通过 Obsidian WikiLink（`[[...]]`）互链，`alwaysUpdateLinks: true` 保证链接自动更新。

### 3.3 插件生态
社区插件清单表明该知识库具备以下扩展能力：
- 数据查询：`dataview`
- 绘图：`obsidian-excalidraw-plugin`
- 看板：`obsidian-kanban`
- 标签管理：`tag-wrangler`
- 模板引擎：`templater-obsidian`
- AI 辅助：`copilot`、`realclaudian`
- 版本控制：`obsidian-git`
- 终端集成：`terminal`
- REST API：`obsidian-local-rest-api`

## 4. 约定与约束

- **附件路径约束**：`app.json` 中 `attachmentFolderPath` 设为 `附件`，意味着所有粘贴图片、下载资源必须进入该目录，否则 Obsidian 无法正确解析相对路径引用。
- **模板目录约束**：`templates.json` 中 `folder` 字段固定为 `模版`，新模板必须放入此目录才能被 Templates/Templater 插件识别。
- **插件启用即生效**：`core-plugins.json` 和 `community-plugins.json` 是 Obsidian 启动时读取的唯一来源——新增插件只需加入对应数组即可启用，无需额外注册表。
- **工作区状态不手工维护**：`workspace.json` 由 Obsidian 自动维护，手动修改可能导致视图错乱，应通过 UI 操作调整布局。
- **无环境变量/密钥管理**：仓库未引入 `.env`、`secrets`、CI 变量等机制；若需敏感信息（如 Git 同步凭据、Copilot 令牌），应依赖 Obsidian 插件自身的设置界面或操作系统级凭据管理器，而非提交到仓库。
- **Git 协作约定**：由于 `obsidian-git` 已启用，整个 `.obsidian/` 目录（含 workspace 快照）会被纳入版本控制，建议通过忽略策略或分支隔离避免多人冲突。

## 5. 总结

该仓库的“配置系统”本质上是 Obsidian 客户端配置的纯声明式 JSON 集合，围绕 `.obsidian/` 目录组织，覆盖应用行为、插件生态、工作区布局与模板路径四个维度。它不是可执行程序的配置框架，而是个人知识库在 Obsidian 中的“环境快照”，适合单人或小团队通过 Git 同步共享。