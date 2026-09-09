# Learning Skill 行为回归场景

这个文件验证 **learner-visible behavior**，不是复制 `learning-skill` 的内部 contract。测试时以 `edge-model-deployment/` 作为真实 workspace，在新会话中安装最新 `learning-skill` 后逐个触发。

每个 Case 只记录四件事：前置、用户怎么说、应该观察到什么、不能产生什么副作用。若 Skill 规则和这里冲突，以 `learning-skill` contract 为准；这里应随真实行为更新，而不是成为第二份规范。

## R01 — Route 是顾问，不是执行器

**前置**：当前 topic 已存在 Map、Records 和母文档。

**用户说**：`$learning-route 我刚刚把“图片如何变成 Input Tensor”这个问题聊明白了，接下来怎么办？`

**期望**：用学习者能懂的话给出下一步建议、简短理由，以及一条可直接复制给 AI 的自然语言 prompt；prompt 携带必要的具体 Record/问题上下文。

**禁止副作用**：不执行 Materialize/Integrate；不要求用户理解 `filling`、`Materialize`、`accepted-existing-material` 等内部词；不 dump 完整状态机。

## R02 — 已有 Map 问题先复用

**用户说**：`我读文章又想到：模型输入为什么需要预处理？这个我想继续弄懂。`

**期望**：识别到 Map 已有等价节点 `模型输入为什么需要预处理？`，回到已有节点/Record/工作路径。

**禁止副作用**：不新增语义重复的 Map Node，不新建重复 Ticket。

## R03 — 同一篇文章里的疑问要分类

**用户说**：`我读 K-001 有三个疑问：Runtime 这里原文没讲清楚；buffer 这个词我只想知道当前含义；Runtime 怎么选择 CPU/GPU/NPU backend 这个我想深入。`

**期望**：分别识别为：原 Record scope 内的 revision；最小 prerequisite；真正可能扩展学习目标的新问题。新问题先 reconcile，只有用户已表达继续研究时才正式进入 Map/待办。

**禁止副作用**：不能把三个问题全部机械建成 Ticket；不能为 tiny prerequisite 建正式 Map Node。

## R04 — 已完成学习直接 Materialize

**用户说**：`刚才关于图片如何变成 Input Tensor 已经讨论清楚了，把它保存成独立学习记录，不要重新研究。`

**期望**：整理已完成理解，生成一个 standalone LearningRecord；若用户明确提供“从 Tensor基础 阅读时产生这个问题”的来源，则 child Record 保存一条 `derived-from`。

**禁止副作用**：不创建 retroactive KnowledgeTicket；不扩大问题范围；不重新做无请求的研究。

## R05 — Revision 保持 Record identity

**用户说**：`Tensor基础里 Tensor / Shape 的定义太抽象，补直接定义和例子，其他正确内容保留。`

**期望**：修改同一路径的 Record，保留 identity、`record_type`、`created_at`、仍成立的 evidence/relations。

**禁止副作用**：不复制新 Record；不为纯 revision 建 Ticket；不顺手扩成新学习主题。

## R06 — Integration 是幂等的

**前置**：`图片如何变成Input Tensor.md` 已与现有 `模型输入为什么需要预处理？` 节点关联并有 completion provenance。

**用户说**：`把这篇接入当前母文档；如果已经接过就只做必要同步。`

**期望**：母文档最多一个准确 callout/index；Map 下最多一个该 Record 的轻量 attachment；已有 completion provenance 不重复；Ticket state 不因 standalone Record 被制造或改变。

**禁止副作用**：不复制正文到母文档；不新增第二个 callout/attachment/completion entry。

## R07 — Knowledge lineage 只有一个真相源

**真实路径**：`Tensor基础` → 阅读时产生 `一个图片是如何被转换成一个 Tensor 的？` → `图片如何变成 Input Tensor`。

**期望**：canonical edge 只存在 child Record 的 `relations[].type: derived-from`；child 正文能看到 `来源脉络`；`knowledge-lineage.md` 可以从 Records 重建左到右图；Map 只显示轻量 Record attachment。

**禁止副作用**：parent Record 不维护 reciprocal backlink；`learning.yaml` 不复制 edge；Map attachment 不变成正式问题节点。

## R08 — `[x]` 必须有完成证据

**操作**：人为设想 Map 有 `[x]`，但对应 node 没有 `map_completions`。

**期望**：status/reconcile 把它视为不一致，不计为完成；若已有材料可能满足，只建议用户明确接受 evidence。

**禁止副作用**：不从聊天记忆或“看起来学过”自动制造 completion provenance。

## R09 — 用户问题数量不受 AI candidate 上限截断

**用户说**：`我读完这篇产生了 6 个问题，帮我分别判断怎么处理。`

**期望**：6 个都被 reconcile/classify；只有 AI 主动推荐但用户尚未接受的问题才受最多 3 个 candidate 的限制。

**禁止副作用**：不能只处理前三个；不能把用户问题全部塞进 `candidate_questions`。

## R10 — `[~]` 只表示真实 partial understanding/work

**期望目标**：`[~]` 应当只在有明确已覆盖部分、同时存在已知剩余缺口时使用；“历史聊过但没有 completion provenance”本身不足以自动证明 partial。

**验证用途**：若后续 Skill contract 收紧 `[~]` 语义，本 Case 用来检查 Map 是否从模糊状态变成可解释状态。

## R11 — Durable Record 是完成的 producer output

**期望目标**：producer handoff 的 LearningRecord 应可独立阅读并达到 branch completion；Record 的持久身份不依赖第二套 `open/in-progress/resolved` 生命周期才能成立。

**验证用途**：用于评估/验证 LearningRecord `status` 元数据是否只是冗余 cache；任何删减都必须保持 Resolve / Materialize / Revise 的 completion behavior。

## R12 — 派生 workflow 状态不能成为第二份真相

**前置**：Map、母文档、Tickets、completion provenance 都可从 workspace 直接读取。

**期望目标**：用户决策（例如 Map 是否确认）可以持久化；可由真实 artifacts 推导的 `stage`、`draft_created`、`next_action` 不应要求每个操作再同步一份缓存。

**验证用途**：状态最小化后，Route/Status 仍能从当前 workspace 推导“现在是什么情况、下一步建议是什么”，而不依赖旧 cache 字段。
