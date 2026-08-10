---
description: 步骤拆解
argument-hint: "<spec name>"
compatibility: 需要 openspec CLI 和 superpower 技能包。
---

对 Spec 需求进行按步骤拆解实现过程

我将为变更输出实现过程的步骤和计划：
- superpower-design.md（详细计划）
- tasks.md（实现状态跟踪）

准备好计划后，运行 /stdd-apply 开始实现

**change-name**：${@:-缺省}

**步骤**

1. **选择变更**

   如果提供了名称，使用它。否则：
   - 如果用户提到了某个变更，从对话上下文中推断
   - 如果只存在一个活动变更，自动选择
   - 如果不明确，运行 `openspec list --json` 获取可用变更，并使用 **ask_user_question tool** 让用户选择

   始终宣布："正在使用变更：<change-name>"以及如何覆盖（例如，`/stdd-plan <other>`）。

2. **检查状态以了解 Schema**

   ```bash
   openspec status --change "<change-name>" --json
   ```
   解析 JSON 以了解：
   - `schemaName`：正在使用的工作流（例如："spec-driven"）
   - 哪个产出物包含任务（对于 spec-driven 通常是 "tasks"，检查其他产出物的状态）

3. **输出细致的实现规划**

   使用 /writing-plans 技能，基于已有文档创建实现计划。

   计划要求：
   - 输出至 openspec/changes/<change-name>/superpower-plan.md
   - 引用设计文档 openspec/changes/<change-name> 下的内容，拆分为可执行任务
   - Plan 文件头必须包含：
   ---
   change: <change-name>
   design-doc: openspec/changes/<change-name>/superpower-design.md
   base-ref: <先运行 git rev-parse HEAD 记录当前提交>
   ---

   输出完成后更新 **todo tool** 中 superpower-plan.md 的进度。

4. **对 superpower-plan.md 进行摘要，并输出为 OpenSpec Tasks**

   - 获取 OpenSpec tasks 指令
      ```bash
      openspec instructions --change "<change-name>" tasks --json
      ```
   - 指令 JSON 包括：
      - `context`：项目背景（对你的约束 - 不要包含在输出中）
      - `rules`：产出物特定规则（对你的约束 - 不要包含在输出中）
      - `template`：用于输出文件的结构
      - `instruction`：此产出物类型的 Schema 特定指导
      - `outputPath`：写入产出物的位置
      - `dependencies`：已完成的产出物，用于读取上下文
   - 读取任何已完成的依赖文件以获取上下文
   - 使用 `template` 作为结构创建产出物文件
   - 应用 `context` 和 `rules` 作为约束 - 但不要将它们复制到文件中
   - 显示简短进度："✓ 已创建 <artifact-id>"

   注意需要保持一一对应 superpower-plan.md 中的任务
   输出完成后更新 **todo tool** 中 tasks.md 的进度。

5. **显示最终状态**

   ```bash
   openspec status --change "<change-name>"
   ```

**输出**

完成所有产出物后，总结：
- 变更名称和位置
- 已创建产出物的列表及简要描述
- 准备就绪："任务输出实现步骤拆解完成，可以开始实现功能。"
- 提示："运行 `/stdd-apply` 开始实现功能。"

**护栏**

- 在创建新产出物之前始终阅读依赖产出物
- 如果上下文极其不清楚，询问用户 - 但倾向于做出合理的决定以保持势头
- 如果同名变更已存在，询问用户是否要继续它或创建一个新的
- 在继续下一个之前，验证写入后每个产出物文件是否存在
