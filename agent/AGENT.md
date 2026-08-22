# AGENT.md —— 对话转笔记 Agent 总指导

本目录是"对话转笔记" agent 工作流的规范中心。目标：对话澄清知识点后，用自然语言（"总结"、"整理"等）触发，自动生成符合仓库规范的 Obsidian 笔记。

## 触发方式

直接在对话中说，例如：
- "把刚才的内容总结一下记到笔记里"
- "整理到 408/数据结构"
- "补充一下线性表笔记里关于循环链表的内容"
- "批量修改：把所有笔记的考点提示改成 callout"

由 note-capture skill 语义识别并执行（位于 `agent\skills\note-capture\`）。

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

## skill 管理约定

- skill 统一存放在 `agent\skills\`，Qoder 已验证可直接扫描加载该目录（无需复制到 `.qoder\skills\`）
- 修改 skill：直接改 `agent\skills\` 下对应文件并提交，Qoder 重新扫描后生效
- 迁移到新机器/新 IDE：复制 `agent\skills\` 对应目录到目标工具的 skill 目录即可
