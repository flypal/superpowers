# 代码质量评审提示模板

派发代码质量评审子智能体时使用本模板。

**目的：** 确认实现质量过硬（清晰、有测试、可维护）

**仅在规格符合性评审通过之后派发。**

```
Task tool (superpowers:code-reviewer):
  使用 requesting-code-review/code-reviewer.md 模板

  WHAT_WAS_IMPLEMENTED: [来自实现者报告]
  PLAN_OR_REQUIREMENTS: 任务 N，来自 [plan-file]
  BASE_SHA: [任务开始前提交]
  HEAD_SHA: [当前提交]
  DESCRIPTION: [任务摘要]
```

**除标准代码质量外，评审还应检查：**
- 每个文件是否职责单一、接口明确？
- 单元是否拆到可独立理解与测试？
- 实现是否遵循计划中的文件结构？
- 本次实现是否新增了已经很大的文件，或显著撑大了已有文件？（不要拿历史文件尺寸说事——聚焦本次变更带来的部分。）

**代码评审返回：** 优点、问题（严重/重要/次要）、评估
