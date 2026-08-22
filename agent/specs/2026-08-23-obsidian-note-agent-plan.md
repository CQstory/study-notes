# Obsidian 对话转笔记 Agent 实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 在 Qoder 中搭建"对话转笔记" agent 工作流：语义触发 → 提取对话要点 → 按仓库规范写入 Obsidian vault。

**Architecture:** 三层结构——`note-capture` skill 负责语义触发与流程编排；文件直写为主通道、mcp-obsidian 为辅助通道；`agent/` 目录作为规范文档与 skill 权威副本备份。设计依据：`agent/specs/2026-08-23-obsidian-note-agent-design.md`。

**Tech Stack:** Qoder Skills（`.qoder/skills/`）、Markdown、PowerShell、git、npx（obsidian-mcp）。

**仓库关键事实（实施者必读）：**
- Vault 根目录：`d:\笔记`，是 git 仓库，提交在工作目录执行 `git add <文件>; git commit -m "..."`
- 现有笔记规范：章节笔记首行 `[[总览|← 返回目录]]`，`# 标题`，`## 一、` 中文编号分节，`> **考点提示**` 引用块，无 frontmatter
- 总览文件：科目目录下同名 md（如 `408\数据结构\数据结构.md`），章节列表格式 `- [[章节名|内容摘要]]`
- 归属目录：408 四科 → `408\操作系统|数据结构|计算机组成原理|计算机网络`；数学 → `数一\`；其他 → vault 根目录
- PowerShell 不支持 `&&`，用 `;` 分隔；复制目录用 `Copy-Item -Recurse`

---

### Task 1: 移植 Trae 的 3 个 Obsidian skill 到 agent/skills/

**Files:**
- Create: `agent\skills\obsidian-markdown\`（含 references 子目录）
- Create: `agent\skills\obsidian-cli\SKILL.md`
- Create: `agent\skills\obsidian-bases\`（含 references 子目录）

- [ ] **Step 1: 复制三个 skill 目录**

```powershell
Copy-Item -Recurse -Force "C:\Users\fll19\.trae-cn\skills\obsidian-markdown" "d:\笔记\agent\skills\obsidian-markdown"
Copy-Item -Recurse -Force "C:\Users\fll19\.trae-cn\skills\obsidian-cli" "d:\笔记\agent\skills\obsidian-cli"
Copy-Item -Recurse -Force "C:\Users\fll19\.trae-cn\skills\obsidian-bases" "d:\笔记\agent\skills\obsidian-bases"
```

- [ ] **Step 2: 验证文件齐全**

```powershell
Get-ChildItem "d:\笔记\agent\skills" -Recurse -File | Select-Object FullName
```

Expected：至少包含
`obsidian-markdown\SKILL.md`、`obsidian-markdown\references\CALLOUTS.md`、`EMBEDS.md`、`PROPERTIES.md`，
`obsidian-cli\SKILL.md`，
`obsidian-bases\SKILL.md`、`obsidian-bases\references\FUNCTIONS_REFERENCE.md`

- [ ] **Step 3: 提交**

```powershell
git -C "d:\笔记" add "agent/skills"; git -C "d:\笔记" commit -m "feat(agent): 移植 Trae 的 3 个 Obsidian skill 作为权威副本"
```

---

### Task 2: 编写 note-capture 核心 skill 权威副本

**Files:**
- Create: `agent\skills\note-capture\SKILL.md`

- [ ] **Step 1: 创建 SKILL.md**

完整内容如下（原样写入，不要增删）：

````markdown
---
name: note-capture
description: 把当前对话中澄清过的知识点整理成 Obsidian 笔记并写入本 vault（d:\笔记），遵循仓库笔记规范（WikiLink、中文编号分节、总览文件链接）。当用户说"总结"、"整理"、"记成笔记"、"记录到"、"归档到笔记"等，或要求修改、补充、批量更新已有笔记时使用。
---

# Note Capture（对话转笔记）

把当前对话提炼为符合仓库规范的 Markdown 笔记，通过文件直写落盘到 vault。用户可能随时手动修改笔记，写入必须遵守"写入安全规则"。

## 工作流程

1. **提取**：从当前对话提炼知识点、解题方法、公式、易错点。数学公式用 LaTeX（行内 `$...$`，块级 `$$...$$`）。忽略寒暄与过程性讨论，只保留结论性知识。
2. **定位**：用 Glob/Grep 在 vault 中搜索同主题笔记（按关键词匹配文件名与正文）。
   - 找到已有笔记 → 合并模式（见写入安全规则）
   - 未找到 → 判断归属目录：408 四科 → `408\操作系统|数据结构|计算机组成原理|计算机网络`；数学 → `数一\`；其他 → vault 根目录。**无法判断时询问用户，不要猜测。**
3. **写入**：
   - 新建笔记用下方"笔记模板"
   - 新建章节级笔记后，在对应科目总览文件（如 `408\数据结构\数据结构.md`）的相应位置追加一行：`- [[笔记名|内容摘要]]`
4. **汇报**：列出新建/修改的文件清单（用 file:// 链接）与各自要点。

## 笔记模板

```markdown
[[总览文件名|← 返回目录]]

# 标题

## 一、第一节

正文。关键术语用**加粗**，操作定义用表格。

> **考点提示**：易错点或常考形式。

---

## 二、第二节

……
```

规则：无 frontmatter；一级分节用 `## 一、二、三` 中文编号；分节间用 `---` 分隔；根目录下的独立小笔记可省略返回目录链接与中文编号。

## 写入安全规则

- **只增不毁**：已有笔记默认在相关章节追加或新增小节；仅当用户明确要求"重写某节"时才替换该节，替换前展示 diff 并等待确认。
- **无标记依赖**：禁止插入任何 agent 专用标记、注释或元数据，输出结构与手写笔记无异。
- **批量修改**：涉及 2 个及以上文件时，先列出变更计划（文件 + 拟改动点），用户确认后逐一执行。

## 错误处理

- 归属目录判断不了 → 询问用户
- 目标笔记结构与模板预期不符 → 降级为文末追加新小节（仍遵守只增不毁）
- mcp-obsidian / npx 不可用 → 不影响流程，文件直写兜底
````

- [ ] **Step 2: 验证**

Read 该文件，确认 frontmatter 的 `name: note-capture` 与 description 完整、正文含"工作流程/笔记模板/写入安全规则/错误处理"四节。

- [ ] **Step 3: 提交**

```powershell
git -C "d:\笔记" add "agent/skills/note-capture"; git -C "d:\笔记" commit -m "feat(agent): 新增 note-capture 对话转笔记 skill 权威副本"
```

---

### Task 3: 安装 skill 到 Qoder 生效目录

**Files:**
- Create: `.qoder\skills\note-capture\`
- Create: `.qoder\skills\obsidian-markdown\`、`.qoder\skills\obsidian-cli\`、`.qoder\skills\obsidian-bases\`

- [ ] **Step 1: 复制权威副本到 .qoder\skills\**

```powershell
New-Item -ItemType Directory -Force "d:\笔记\.qoder\skills" | Out-Null
Copy-Item -Recurse -Force "d:\笔记\agent\skills\note-capture" "d:\笔记\.qoder\skills\note-capture"
Copy-Item -Recurse -Force "d:\笔记\agent\skills\obsidian-markdown" "d:\笔记\.qoder\skills\obsidian-markdown"
Copy-Item -Recurse -Force "d:\笔记\agent\skills\obsidian-cli" "d:\笔记\.qoder\skills\obsidian-cli"
Copy-Item -Recurse -Force "d:\笔记\agent\skills\obsidian-bases" "d:\笔记\.qoder\skills\obsidian-bases"
```

- [ ] **Step 2: 验证生效**

```powershell
Get-ChildItem "d:\笔记\.qoder\skills" -Directory | Select-Object Name
```

Expected：`note-capture`、`obsidian-bases`、`obsidian-cli`、`obsidian-markdown` 四个目录。
注：Qoder 的技能列表可能需要重新加载窗口后才显示，文件就位即完成本任务。

- [ ] **Step 3: 提交**

```powershell
git -C "d:\笔记" add ".qoder"; git -C "d:\笔记" commit -m "feat(agent): 安装 4 个 skill 到 .qoder/skills 生效目录"
```

---

### Task 4: 编写 AGENT.md 总指导

**Files:**
- Create: `agent\AGENT.md`

- [ ] **Step 1: 创建 AGENT.md**

完整内容如下：

````markdown
# AGENT.md —— 对话转笔记 Agent 总指导

本目录是"对话转笔记" agent 工作流的规范中心。目标：对话澄清知识点后，用自然语言（"总结"、"整理"等）触发，自动生成符合仓库规范的 Obsidian 笔记。

## 触发方式

直接在对话中说，例如：
- "把刚才的内容总结一下记到笔记里"
- "整理到 408/数据结构"
- "补充一下线性表笔记里关于循环链表的内容"
- "批量修改：把所有笔记的考点提示改成 callout"

由 note-capture skill 语义识别并执行（位于 `.qoder\skills\note-capture\`）。

## 目录约定

| 内容 | 位置 |
|---|---|
| 408 四科笔记 | `408\操作系统`、`408\数据结构`、`408\计算机组成原理`、`408\计算机网络` |
| 数学一笔记 | `数一\` |
| 其他零散笔记 | vault 根目录 |
| 科目总览 | 各科目目录下同名 md（如 `408\数据结构\数据结构.md`） |

## 笔记规范摘要

- 章节笔记首行 `[[总览|← 返回目录]]`；`# 标题`；`## 一、` 中文编号分节；`---` 分隔；`> **考点提示**` 引用块；无 frontmatter
- 总览文件条目格式：`- [[笔记名|内容摘要]]`
- 数学公式用 LaTeX
- 详见 `skills\obsidian-markdown\SKILL.md`

## 写入安全规则

1. 只增不毁：默认追加；替换已有内容前必须展示 diff 并经用户确认
2. 不插入任何 agent 专用标记
3. 批量修改（≥2 文件）先列变更计划，确认后执行

## 工具通道

- **主通道**：文件直写（Qoder Read/Write/SearchReplace 工具），始终可用
- **辅助通道**：mcp-obsidian MCP（search-vault、add-tags 等），配置见 `specs\2026-08-23-obsidian-note-agent-design.md` 第 3 节；不可用时不影响主流程
- **备用**：Obsidian CLI（`D:\Obsidian\Obsidian.com`，需主程序运行），本期不启用

## skill 同步约定

- 权威副本在 `agent\skills\`；生效副本在 `.qoder\skills\`
- 修改 skill 时：先改 `agent\skills\`，再整目录覆盖复制到 `.qoder\skills\`，两处一起提交
````

- [ ] **Step 2: 验证**：Read 该文件，确认五个小节齐全。

- [ ] **Step 3: 提交**

```powershell
git -C "d:\笔记" add "agent/AGENT.md"; git -C "d:\笔记" commit -m "docs(agent): 添加 AGENT.md 总指导"
```

---

### Task 5: 编写备份指导

**Files:**
- Create: `agent\backup\备份指导.md`

- [ ] **Step 1: 创建备份指导.md**

完整内容如下：

````markdown
# 备份指导

## 日常备份（推荐）：obsidian-git

vault 已装 obsidian-git 插件且本目录是 git 仓库：

1. Obsidian 中打开命令面板（Ctrl+P）→ 输入 `Git`
2. 常用命令：`Git: Commit all changes`、`Git: Push`
3. 建议在插件设置中开启自动 commit + push 定时任务
4. 需在 git 仓库配置远程地址：`git remote add origin <你的仓库地址>`（若尚未配置）

## 手动全量备份

复制整个 vault 目录到外部存储：

```powershell
Copy-Item -Recurse "d:\笔记" "E:\backup\笔记-$(Get-Date -Format yyyyMMdd)"
```

## agent 资产恢复

- `agent\skills\` 是 skill 权威副本；`.qoder\skills\` 是生效副本
- 若 `.qoder\skills\` 丢失或损坏：整目录从 `agent\skills\` 复制覆盖即可
- 若 skill 需要在新机器/新 IDE 复用：直接复制 `agent\skills\` 对应目录到目标工具的 skill 目录

## 注意

- 提交前确认没有误删笔记：`git status` 中 Deleted 条目要逐一核实
- obsidian-git 的提交与 agent 的提交共用同一仓库，历史不会冲突
````

- [ ] **Step 2: 验证**：Read 该文件，确认四节齐全。

- [ ] **Step 3: 提交**

```powershell
git -C "d:\笔记" add "agent/backup"; git -C "d:\笔记" commit -m "docs(agent): 添加备份指导"
```

---

### Task 6: 配置 mcp-obsidian 到 Qoder

**Files:**
- 无文件改动（Qoder MCP 设置通过 UI 添加）

- [ ] **Step 1: 验证 npx 环境可用**

```powershell
npx --version
```

Expected：输出版本号。若失败，记录为"辅助通道不可用"，主流程不受影响，跳到 Step 3。

- [ ] **Step 2: 引导用户在 Qoder MCP 设置中添加服务器**

请用户打开 Qoder 的 MCP 设置（设置 → MCP → 添加服务器），粘贴以下配置：

```json
{
  "mcp-obsidian": {
    "command": "npx",
    "args": ["-y", "obsidian-mcp", "D:/笔记"]
  }
}
```

Expected：Qoder 的 MCP 服务器列表中出现 mcp-obsidian 且状态正常。此步骤需要用户操作 UI，agent 无法代劳。

- [ ] **Step 3: 记录结果**

在 Task 7 的冒烟测试汇报中注明 MCP 是否配置成功。无需提交。

---

### Task 7: 冒烟测试——生成测试笔记

**Files:**
- Create: `agent\Obsidian集成资产检测.md`（测试笔记，放 agent 目录不污染知识库）
- Modify: `agent\AGENT.md` 不需要更新；本任务不修改任何科目总览

- [ ] **Step 1: 按 note-capture 流程创建测试笔记**

创建 `agent\Obsidian集成资产检测.md`，内容如下（这是本次对话真实结论，遵循笔记规范；agent 目录笔记省略返回目录链接）：

````markdown
# Obsidian 集成资产检测

## 一、Qoder 侧

Qoder 本机无 Obsidian 专属 MCP 与 skill，但具备 vault 文件直接读写能力，作为笔记写入主通道。

## 二、Vault 侧（d:\笔记）

| 插件 | 作用 |
|---|---|
| `obsidian-local-rest-api` | HTTP API（HTTPS 27124），可供外部工具操作笔记 |
| `realclaudian` | Obsidian 内运行 Claude 会话 |
| `obsidian-git` | 版本管理与同步 |

其他：`templater-obsidian`、`dataview`、`folder-notes`、`obsidian-kanban`、`terminal` 等。

## 三、Trae CN 可移植资产

- 3 个 skill：`obsidian-cli`、`obsidian-markdown`、`obsidian-bases`（已移植至本目录 `skills\`）
- MCP 配置：`npx -y obsidian-mcp "D:/笔记"`（文件系统直操作，无需 Obsidian 运行）
- 本机 Obsidian CLI：`D:\Obsidian\Obsidian.com`（备用通道）

> **考点提示**：本文为 agent 冒烟测试产物，确认集成链路正常后可保留作资产清单或删除。
````

- [ ] **Step 2: 验证**

```powershell
Get-Content "d:\笔记\agent\Obsidian集成资产检测.md" -TotalCount 3
```

Expected：前两行为空行/`# Obsidian 集成资产检测`。并在 Obsidian 中确认该笔记渲染正常（用户目检）。

- [ ] **Step 3: 提交并汇报**

```powershell
git -C "d:\笔记" add "agent/Obsidian集成资产检测.md"; git -C "d:\笔记" commit -m "test(agent): 冒烟测试笔记——Obsidian 集成资产检测"
```

汇报内容：全部交付物清单（对照设计文档第 8 节逐项确认）、MCP 配置状态、建议用户在新会话中说"整理一下这次对话"验证 skill 语义触发。

---

## 自检记录

- **Spec 覆盖**：设计文档第 3 节布局 → Task 1/2/4/5；第 4 节工作流 → Task 2（skill 正文）；第 5 节安全规则 → Task 2/4；第 6 节错误处理 → Task 2；MCP 配置 → Task 6；第 7 节测试 → Task 7；第 8 节交付物 6 项均有对应任务
- **占位符扫描**：无 TBD/TODO；所有文档内容完整给出
- **一致性**：skill 统一存放于 `agent\skills\`（见下方执行偏离记录）

## 执行偏离记录（2026-08-23）

- **Task 3 取消**：执行中发现 Qoder 直接扫描加载 `agent\skills\` 目录（当前会话技能列表可见全部 4 个 skill，base directory 即 `d:\笔记\agent\skills\...`），因此无需复制到 `.qoder\skills\`，避免同名 skill 重复。AGENT.md 与备份指导已按单一位置方案编写。
- **Task 6 部分完成**：npx 环境验证通过（v11.6.2）；MCP 服务器需用户在 Qoder 设置面板手动添加，已提供配置片段。
