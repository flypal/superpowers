---
name: requesting-code-review
description: 在完成任务、实现重要功能或合并到 main 之前使用，以确认工作符合需求
---

# 请求代码评审

派发 superpowers:code-reviewer 子智能体，在问题蔓延前发现问题。评审者会获得精心准备的上下文——绝不会是你的会话历史。这样评审者能聚焦产物，而非你的思路，也为你保留上下文以继续工作。

**核心原则：** 早评审、常评审。

## 何时请求评审

**必须：**
- subagent-driven 开发中每个任务之后  
- 完成重要功能之后  
- 合并到 main 之前  

**可选但很有价值：**
- 卡住时（新视角）  
- 重构前（基线检查）  
- 修复复杂 bug 之后  

## 如何请求

**1. 获取 git SHA：**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # 或 origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. 派发 code-reviewer 子智能体：**

使用 Task 工具，类型为 superpowers:code-reviewer，按 `code-reviewer.md` 模板填写。

**占位符：**
- `{WHAT_WAS_IMPLEMENTED}` — 你刚做了什么  
- `{PLAN_OR_REQUIREMENTS}` — 应该达成什么  
- `{BASE_SHA}` — 起始提交  
- `{HEAD_SHA}` — 结束提交  
- `{DESCRIPTION}` — 简短摘要  

**3. 处理反馈：**
- 严重问题立即修  
- 重要问题在继续前修  
- 次要问题记下稍后处理  
- 若评审有误，有理有据地反驳  

## 示例

```
[刚完成任务 2：添加校验函数]

你：在继续前先请求代码评审。

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[派发 superpowers:code-reviewer 子智能体]
  WHAT_WAS_IMPLEMENTED: 会话索引的校验与修复函数
  PLAN_OR_REQUIREMENTS: docs/superpowers/plans/deployment-plan.md 中的任务 2
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661
  DESCRIPTION: 新增 verifyIndex() 与 repairIndex()，覆盖 4 类问题

[子智能体返回]：
  优点：架构清晰、有真实测试
  问题：
    重要：缺少进度指示
    次要：上报间隔的魔数 100
  评估：可以继续

你：[补上进度指示]
[进入任务 3]
```

## 与工作流整合

**子智能体驱动开发：**
- **每**个任务后评审  
- 在问题叠加前抓住  
- 修完再进入下一任务  

**执行计划：**
- 每批（3 个任务）后评审  
- 吸收反馈再继续  

**临时开发：**
- 合并前评审  
- 卡住时评审  

## 危险信号

**绝不：**
- 因为「很简单」就跳过评审  
- 忽略严重问题  
- 重要问题未修就继续  
- 对合理技术反馈抬杠  

**若评审有误：**
- 用技术理由反驳  
- 用代码/测试证明可行  
- 请求澄清  

模板见：`requesting-code-review/code-reviewer.md`
