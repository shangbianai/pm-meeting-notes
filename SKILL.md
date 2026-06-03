---
name: pm-transcript-organizer
description: Use this skill when the user provides long Chinese transcript-like text, meeting notes, copied speech-to-text content, work discussions, brainstorming records, or asks to summarize meeting minutes, product requirements, business processes, todos, or product optimization items from such material. It defaults to generating structured meeting minutes when the user only provides long raw text, then offers a fixed numbered follow-up menu.
---

# PM Transcript Organizer

## Purpose

Turn raw Chinese speech-to-text material into reliable product-manager-facing outputs. The source is often messy: meeting transcripts, work records, business discussions, brainstorming, or fragmented operational notes.

The assistant must preserve meaning, remove noise, organize logic, and distinguish original content from inferred product analysis.

## Input Recognition

First classify the user input:

- **Raw transcript only**: long, oral, fragmented, contains repeated phrases, filler words, speaker-like turns, or meeting/work discussion content. Default to task `1. 会议纪要`.
- **Raw transcript plus instruction**: execute the requested task directly.
- **Short normal question**: answer normally; do not force the meeting workflow.
- **Ambiguous long text**: treat as raw transcript and produce default meeting minutes, then offer the fixed numbered menu.

If a file is uploaded, read/extract the content first. If the file is very long, process by topic clusters before producing the final output.

## Fixed Task Menu

Always use this exact numbering when offering follow-up actions:

1. **会议纪要**: default output for raw transcript; produce formal meeting minutes.
2. **快速梳理**: concise structured overview when the user's focus is unclear.
3. **梳理产品需求**: convert the material into a detailed PRD-style document.
4. **梳理业务流程**: produce business process explanation plus Mermaid flowchart code.
5. **梳理待办**: extract actionable tasks with owner, deadline, priority, dependencies, and status.
6. **梳理产品优化项**: extract feature improvements, bug fixes, UX upgrades, iteration items, and validation suggestions.

When the user replies with only a number, apply that numbered task to the latest transcript/material in the conversation.

## Default Behavior

When the user only drops a long transcript, do not ask what to do first. Produce `1. 会议纪要` immediately.

End the output with:

```text
你也可以继续回复序号让我进一步处理：
2 快速梳理｜3 梳理产品需求｜4 梳理业务流程｜5 梳理待办｜6 梳理产品优化项
```

If the transcript is too incomplete to produce useful minutes, still provide a lightweight meeting-minutes draft and clearly list missing information under `待确认问题`.

## Transcript Cleanup Rules

- Remove filler words, repeated phrases, obvious transcription noise, and irrelevant small talk.
- Keep names, roles, dates, systems, features, business terms, decisions, numbers, and dependencies.
- Merge scattered statements about the same topic.
- Preserve the original meaning; do not over-polish into claims the transcript does not support.
- If speaker attribution is unclear, use neutral wording such as `材料中提到`.
- If information is absent, write `未明确`, `未提及`, or `需确认`.
- Mark uncertain product judgments as `推测` or `建议`, not as facts.

## Source vs Inference

Separate content into three levels whenever useful:

- **原始信息**: what the transcript directly says, preserving the speaker's intent.
- **结构化理解**: organized interpretation of the original content.
- **产品判断/建议**: PM analysis, hypotheses, risks, or recommendations inferred from the material.

Do not invent owners, deadlines, priorities, metrics, or decisions. If a suggested value is useful, label it as suggested.

## Task Templates

### 1. 会议纪要

Use this as the default structure:

```markdown
**会议纪要**

**一、会议基本信息**
- 会议主题：
- 会议时间：
- 参会人员：
- 会议背景：

**二、会议结论摘要**
- 

**三、核心讨论内容**
1. 议题一：
   - 原始信息：
   - 结构化理解：
   - 结论/影响：

**四、关键决策与共识**
| 决策/共识 | 来源依据 | 影响范围 | 待确认 |
|---|---|---|---|

**五、问题、风险与阻塞**
| 问题/风险 | 影响 | 当前判断 | 建议处理 |
|---|---|---|---|

**六、行动项**
| 事项 | 责任人 | 截止日期 | 优先级 | 备注 |
|---|---|---|---|---|

**七、待确认问题**
- 
```

If the material is not a formal meeting but is still a work discussion, adapt headings naturally while keeping the same substance.

### 2. 快速梳理

Use when the user asks `快速梳理` or when they need a brief structured understanding:

```markdown
**快速梳理**

**一句话结论**

**核心背景**

**主要讨论点**

**已经明确的事项**

**未明确/需确认**

**建议下一步**
```

### 3. 梳理产品需求

Produce a PRD-style document. Include at least:

```markdown
**产品需求文档 PRD**

**一、项目/产品背景**
**二、目标用户与使用场景**
**三、原始信息梳理**
- 保留与需求相关的原始观点、领导/同事/客户表达、业务约束。

**四、需求抽象与问题定义**
**五、产品目标与成功指标**
**六、产品范围**
- 本期范围
- 暂不处理

**七、详细功能需求**
| 模块 | 用户故事/场景 | 功能描述 | 规则 | 优先级 | 验收标准 |
|---|---|---|---|---|---|

**八、业务流程**
**九、页面/模块建议**
**十、数据与埋点建议**
**十一、权限、异常与边界情况**
**十二、风险与依赖**
**十三、待办事项**
**十四、待确认问题**
```

Prioritize PM usefulness over decorative completeness. Keep unsupported sections concise and mark missing information.

### 4. 梳理业务流程

Output exactly two main parts:

- `一、业务流程描述`
- `二、Mermaid 流程图`

The second part must include a valid `mermaid` fenced code block, for example:

````markdown
```mermaid
flowchart TD
  A["开始"] --> B["关键步骤"]
```
````

Include roles, triggers, decision points, normal path, exception path, and final outcomes when present.

### 5. 梳理待办

Extract actionable work items:

```markdown
**待办事项**

| 序号 | 待办事项 | 背景/来源 | 责任人 | 协作方 | 截止日期 | 优先级 | 状态 | 依赖/风险 |
|---|---|---|---|---|---|---|---|---|
```

Rules:

- `责任人` and `截止日期` must be `未明确` if absent.
- Use `建议P0/P1/P2` only when priority is inferred.
- Separate real action items from open questions.

### 6. 梳理产品优化项

Focus on improvements and iteration:

```markdown
**产品优化项**

| 序号 | 类型 | 问题/机会 | 原始依据 | 影响 | 优化建议 | 建议优先级 | 责任人 | 时间 | 验证指标 |
|---|---|---|---|---|---|---|---|---|---|
```

Types may include:

- 功能优化
- Bug 修复
- 体验升级
- 流程优化
- 权限/配置完善
- 数据/埋点完善
- 后台/运营能力
- 其他迭代项

## Network Search

Use web search only when the user asks for external verification, the transcript references current facts that may have changed, or external context is required to avoid inaccurate analysis. Cite sources when web search is used.

## Style

- Write in Chinese unless the user requests otherwise.
- Be structured, decisive, and PM-oriented.
- Do not over-summarize away important original signals.
- Prefer tables for tasks, decisions, risks, owners, deadlines, and optimization items.
- Ask concise clarification questions only when a critical ambiguity would change the output materially.
