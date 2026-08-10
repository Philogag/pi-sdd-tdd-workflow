---
description: 实现 OpenSpec 变更中的任务
argument-hint: "<spec name>"
compatibility: 需要 openspec CLI 和 superpower 技能包。
---

实现 OpenSpec 变更中的任务。

**change-name**：${@:-缺省}

**步骤**

1. **选择变更**

   如果提供了名称，使用它。否则：
   - 如果用户提到了某个变更，从对话上下文中推断
   - 如果只存在一个活动变更，自动选择
   - 如果不明确，运行 `openspec list --json` 获取可用变更，并使用 **ask_user_question tool** 让用户选择

   始终宣布："正在使用变更：<change-name>"以及如何覆盖（例如，`/opsx-apply <other>`）。

2. **检查状态以了解 Schema**
   ```bash
   openspec status --change "<change-name>" --json
   ```
   解析 JSON 以了解：
   - `schemaName`：正在使用的工作流（例如："spec-driven"）
   - 哪个产出物包含任务（对于 spec-driven 通常是 "tasks"，检查其他产出物的状态）

3. **获取应用指令**

   ```bash
   openspec instructions apply --change "<name>" --json
   ```

   这返回：
   - 上下文文件路径（因 Schema 而异）
   - 进度（总计，完成，剩余）
   - 带有状态的任务列表
   - 基于当前状态的动态指令

   **处理状态：**
   - 如果 `state: "blocked"`（缺少产出物）：显示消息，建议使用 `/opsx:continue`
   - 如果 `state: "all_done"`：祝贺，建议归档
   - 否则：继续实现

4. **阅读上下文文件**

   阅读 apply instructions 输出中 `contextFiles` 列出的文件。
   文件取决于正在使用的 Schema：
   - **spec-driven**: proposal, specs, design, tasks
   - 其他模式：遵循 CLI 输出中的 contextFiles

5. **显示当前进度**

   使用 **todo tool** 实时向用户反馈 tasks 进度

6. **检查工作分支**

   检查是否存在开发分支 `feat/<change-name>`
   如果不存在则遵循 skill `/using-git-worktrees` 进行初始化
   优先使用git worktree拆分工作区

7. **开始实现任务**

   对于整个spec tasks
   - 优先使用 skill /subagent-driven-development 推进任务实现
   - 尽可能并行使地实现
   - 更新 **todo tool** 状态

   对于每个待处理任务：
   - 进行所需的代码更改
   - 保持更改最小化且专注
   - 向开发分支 `feat/<change-name>` 进行提交
   - 在主工作区 tasks.md 中标记任务完成：`- [ ]` → `- [x]`
   - 更新 **todo tool** 状态
   - 继续下一个任务

   **暂停如果：**
   - 任务不清楚 → 询问澄清
   - 实现揭示了设计问题 → 建议更新产出物
   - 遇到错误或阻碍 → 报告并等待指导
   - 用户中断

8. **清理 todo tool**

**完成时的输出**

```
## 实现完成

**变更：** <change-name>
**Schema：** <schema-name>
**进度：** 7/7 任务已完成 ✓

### 本次会话已完成
- [x] 任务 1
- [x] 任务 2
...

所有任务已完成！您可以使用 `/stdd-archive` 收尾此变更。
```

**护栏**

- 继续执行任务直到完成或受阻
- 开始前始终阅读上下文文件（来自 apply instructions 输出）
- 如果任务模棱两可，暂停并在实现前询问
- 如果实现揭示了问题，暂停并建议更新产出物
- 保持代码更改最小化并限定在每个任务范围内
- 完成每个任务后立即更新任务复选框
- 遇到错误、阻碍或不清楚的需求时暂停 - 不要猜测
- 使用 CLI 输出中的 contextFiles，不要假设特定的文件名

**流畅的工作流集成**

此技能支持"变更上的操作"模型：

- **可以随时调用**：在所有产出物完成之前（如果存在任务），部分实现之后，与其他操作交错
- **允许产出物更新**：如果实现揭示了设计问题，建议更新产出物 - 不是阶段锁定的，流畅地工作
