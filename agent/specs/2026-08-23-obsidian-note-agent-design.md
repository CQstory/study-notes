# Obsidian 对话转笔记 Agent 设计

日期：2026-08-23
状态：已获用户确认（方案 A：Skill 规范 + 直接文件写入）

## 1. 背景与目标

本仓库（`d:\笔记`）是由 Obsidian 管理的 408 计算机考研 + 数学一复习笔记库。目标：在 Qoder 中搭建一套 agent 工作流，在对话澄清知识点后，用户以自然语言（"总结"、"整理"等字眼）触发，自动把对话内容转换为符合仓库规范的笔记。

### 前期检测结论

- Qoder 侧：无 Obsidian 专属 MCP/skill；但有 vault 文件直接读写能力（主通道）
- Vault 侧：已装 `obsidian-local-rest-api`、`realclaudian`、`obsidian-git` 等插件
- 本机 Trae CN（`C:\Users\fll19\.trae-cn`）存在三个可移植 skill：`obsidian-cli`、`obsidian-markdown`、`obsidian-bases`
- Trae 中已跑通的 MCP 配置：`npx -y obsidian-mcp "D:/笔记"`（文件系统直操作，无需 Obsidian 运行）
- 本机 Obsidian CLI 位于 `D:\Obsidian\Obsidian.com`（备用通道，本期不使用）

## 2. 总体架构

三层结构：

1. **触发与编排层**：`note-capture` 核心 skill，语义识别"总结/整理/记录笔记"等触发词，编排整个流程
2. **执行层**：文件直写主通道（Qoder 原生 Read/Write/SearchReplace/Glob/Grep 工具）+ mcp-obsidian 辅助通道（搜索 vault、批量打标签）
3. **规范与备份层**：`agent/` 目录存放 AGENT.md、skill 权威副本、备份指导

被否决的替代方案：纯 MCP 驱动（工具粒度粗，合并/批量不灵活）、Obsidian CLI 驱动（依赖主程序运行）。

## 3. 文件布局

```
d:\笔记\agent\
├── AGENT.md                  # 总指导：触发词、工作流程、写入规范、批量修改规则
├── skills\                   # skill 权威副本（备份 + 可读文档）
│   ├── note-capture\SKILL.md        # ★ 核心：对话转笔记（新建）
│   ├── obsidian-markdown\…          # 从 Trae 移植（含 references）
│   ├── obsidian-cli\…               # 从 Trae 移植
│   └── obsidian-bases\…             # 从 Trae 移植（含 references）
├── backup\备份指导.md         # 备份与迁移说明（含 obsidian-git 提示）
└── specs\                    # 设计文档
```

- skill 生效位置：Qoder 识别的项目 skill 目录（实施时按 `create-skill` 规范确定），与 `agent/skills/` 权威副本保持同步
- mcp-obsidian 配置（加入 Qoder MCP 设置）：

```json
{
  "mcp-obsidian": {
    "command": "npx",
    "args": ["-y", "obsidian-mcp", "D:/笔记"]
  }
}
```

## 4. 工作流程（note-capture）

1. **触发**：用户消息含"总结 / 整理 / 记成笔记 / 记录到…"等意图词语义激活，可附带指向（如"整理到 408/数据结构"）
2. **提取**：从当前对话提炼知识点、解题方法、公式、易错点；数学公式用 LaTeX
3. **定位**：Glob/Grep 搜索 vault 是否已有同主题笔记
   - 已有 → 合并模式
   - 没有 → 判断归属目录（408 四科子目录 / 数一 / 根目录），无法判断时询问用户
4. **写入**：
   - 新建笔记格式：首部 `[[总览|← 返回目录]]` + `# 标题`，`## 一、` 中文编号分节，`> **考点提示**` 引用块，无 frontmatter（与现有风格一致）
   - 新建章节级笔记时，同步在对应总览文件追加 `- [[笔记名|内容摘要]]`
5. **汇报**：列出新建/修改的文件清单及要点

## 5. 写入安全规则

用户会手动修改/增补笔记，也可能要求批量修改，因此：

- **只增不毁**：已有笔记默认在相关章节追加或新增小节；仅当明确"重写某节"时才替换该节，替换前展示 diff
- **无标记依赖**：不插入 agent 专用标记/注释，内容结构与手写笔记无异，用户手改后 agent 下次仍能正常合并
- **批量修改**：涉及多文件时先列出变更计划（文件 + 拟改动点），用户确认后逐一执行

## 6. 错误处理

| 场景 | 处理 |
|---|---|
| 归属目录判断不了 | 询问用户 |
| 目标笔记结构与预期不符 | 降级为文末追加新小节 |
| npx / MCP 不可用 | 不影响主流程，文件直写兜底 |
| 替换已有内容 | 先展示 diff，确认后执行 |

## 7. 测试计划

实施完成后端到端冒烟测试：把本次对话中"Trae Obsidian 资产检测"结论整理成一篇测试笔记，验证文件生成、WikiLink 正确性、总览文件更新、Obsidian 渲染；确认无误后可删除或保留。

## 8. 交付物清单

1. `agent/AGENT.md` —— 总指导文档
2. `agent/skills/note-capture/SKILL.md` —— 核心 skill（并安装到 Qoder 生效位置）
3. `agent/skills/obsidian-markdown|obsidian-cli|obsidian-bases` —— Trae skill 移植副本（按需安装）
4. `agent/backup/备份指导.md` —— 备份与迁移指导
5. Qoder MCP 设置中的 mcp-obsidian 服务器配置
6. 冒烟测试笔记一篇
