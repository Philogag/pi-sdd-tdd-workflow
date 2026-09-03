---
description: 收尾提案变更
argument-hint: "<spec name>"
compatibility: 需要 openspec CLI 和 superpower 技能包。
---

归档实验性工作流中已完成的变更，合并开发分支中的代码并提交到主工作区。

**change-name**：$@（未提供时按步骤 1 推断）

**步骤**

1. **选择变更**

   如果提供了名称，使用它。否则：
   - 如果用户提到了某个变更，从对话上下文中推断
   - 如果只存在一个活动变更，自动选择
   - 如果不明确，运行 `openspec list --json` 获取可用变更，并使用 **ask** tool 让用户选择

   始终宣布："正在使用变更：<change-name>"以及如何覆盖（例如，`/stdd-archive <other>`）。

2. **检查产出物完成状态**

   运行 `openspec status --change "<change-name>" --json` 检查产出物完成情况。

   解析 JSON 以了解：
   - `schemaName`：正在使用的工作流
   - `artifacts`：产出物列表及其状态（`done` 或其他）

   **如果有任何产出物未 `done`：**
   - 显示列出未完成产出物的警告
   - 提示用户确认是否继续
   - 如果用户确认，则继续

3. **检查任务完成状态**

   阅读任务文件（通常是 `tasks.md`）以检查未完成的任务。

   统计标记为 `- [ ]`（未完成）与 `- [x]`（已完成）的任务。

   **如果发现未完成的任务：**
   - 显示警告，显示未完成任务的数量
   - 提示用户确认是否继续
   - 如果用户确认，则继续

   **如果没有任务文件存在：** 继续，无需任务相关警告。

4. **执行归档**

   如果归档目录不存在，则创建它：
   ```bash
   mkdir -p openspec/changes/archive
   ```

   使用当前日期生成目标名称：`YYYY-MM-DD-<change-name>`

   **检查目标是否已存在：**
   - 如果是：失败并报错，建议重命名现有归档或使用不同日期
   - 如果否：将变更目录移动到归档

   ```bash
   mv openspec/changes/<change-name> openspec/changes/archive/YYYY-MM-DD-<change-name>
   ```

5. **压缩开发分支，并提交到主工作区**

   将全部修改点压缩为一个 commit 进行提交到主工作区
   - 开发分支 `feat/<change-name>` 中的所有修改
   - 当前 change 的归档目录

   代码格式化
   - 应当对提交的代码进行格式化
   - 遵循仓库已有格式化配置

   对于 commit message 格式，按以下顺序选择
   - 当前工作区 submodule 的 .gitmessage
   - 当前工作区 submodule 的 5 条历史提交格式总结
   - 根仓库的 .gitmessage
   - 根仓库的 5 条历史提交格式总结

**成功时的输出**

```
## 归档完成

**变更：** <change-name>
**Schema：** <schema-name>
**归档至：** openspec/changes/archive/YYYY-MM-DD-<name>/
**规范：** ✓ 已同步到主规范
**提交：** <git commit short hash>

所有产出物已完成。所有任务已完成。
<提示用户推送代码>
```

**错误时的输出（归档已存在）**

```
## 归档失败

**变更：** <change-name>
**目标：** openspec/changes/archive/YYYY-MM-DD-<name>/

目标归档目录已存在。

**选项：**
1. 重命名现有归档
2. 如果是重复的，删除现有归档
3. 等待不同的日期再归档
```

**防护措施**
- 如果未提供变更，始终提示选择
- 使用产出物图（openspec status --json）进行完成度检查
- 不要在警告时阻止归档 - 只需告知并确认
- 移动到归档时保留 .openspec.yaml（它与目录一起移动）
- 显示清晰的操作摘要
- 如果请求同步，读取 `skill://openspec-sync-specs`（若已安装）并按其中流程执行（代理驱动）
- 如果存在增量规格说明，请始终运行同步评估，并在提示前显示综合摘要
