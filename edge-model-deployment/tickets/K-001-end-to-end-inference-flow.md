---
version: 2
id: K-001
title: 端侧程序如何把模型部署各环节串起来？
status: open

topic:
  id: edge-model-deployment
  map_node: end-to-end-flow

question: >
  一个训练完成的模型进入真实端侧程序后，模型文件、Runtime、输入数据、预处理、Input Tensor、Forward/Inference、Output Tensor、后处理和业务结果之间，究竟按照怎样的执行链路连接起来？

why_needed: >
  这是当前 Learning Map 中唯一尚未完成的正式节点。解决它的目的，是把已经学习过的 Tensor、Forward、训练与推理、模型文件、预处理和后处理第一次串成一条完整部署主线，从而能够阅读真实端侧部署模块的主流程。

gap_type: concept

result: null
---

# K-001 — 端侧程序如何把模型部署各环节串起来？

## 要做什么

用一条典型端侧推理调用链，把模型文件、Runtime、原始业务输入、预处理、Input Tensor、Inference、Output Tensor、后处理和最终业务结果串起来，并解释每一步为什么存在、接收什么、产出什么，以及它与前后步骤的职责边界。

## 范围

- 只解释一条典型的端侧推理主链路。
- 明确模型文件、Runtime、输入/输出 Tensor、预处理、Inference、后处理各自扮演的角色。
- 说明初始化阶段与“每次推理”阶段的区别，例如模型通常不是每帧都重新加载。
- 解释到足以让读者面对真实部署代码时，能够判断某段代码大致属于加载、预处理、推理还是后处理。

不展开以下内容：

- Runtime 内部算子调度机制。
- ONNX、TFLite、MNN、NCNN 等具体框架之间的差异。
- 模型格式转换过程。
- 量化原理与校准。
- NPU、Delegate、Execution Provider 等硬件加速内部实现。
- 训练阶段的反向传播与优化算法。

## 可用材料

- 当前项目已经完成的 Tensor、shape、dtype、batch 学习内容。
- 当前项目已经完成的 Forward / Inference 学习内容。
- 当前项目已经完成的训练与推理差异学习内容。
- 当前项目已经完成的模型文件、预处理、后处理学习内容。

这些材料用于承接已有概念，不要求逐条复述；结果应围绕本 Ticket 的端到端主线重新组织。

## 证据与表达要求

- 这是概念 Ticket，不要求绑定某个真实代码仓库。
- 对通用端侧推理流程中的稳定事实，可直接作为概念解释。
- 对“不同 Runtime/模型可能不同”的实现差异要明确标出，不把一种框架的 API 形式误写成所有框架的固定流程。
- 重点说明机制和数据流，不扩展成完整端侧部署课程。

## 需要回答

1. 模型文件为什么不能直接接收一张图片并自动给出业务结果？Runtime 在中间承担什么角色？
2. 原始业务数据为什么必须先经过预处理才能进入模型？预处理最终应该得到什么？
3. Input Tensor 交给 Runtime 后，所谓 `Inference/Forward` 实际完成了什么？
4. Runtime 为什么输出的仍然通常是 Tensor，而不是“检测到了猫”“识别为某人”这样的业务语义？
5. 后处理如何把 Output Tensor 转换成业务程序真正能够使用的数据？
6. 初始化阶段与每一次推理阶段分别做哪些事情？
7. 如果以后阅读一段真实部署代码，应该怎样快速判断它位于整条链路的哪个位置？

## 交付与验收

使用 `learning-note` 产出一个 `record_type: note` 的 LearningRecord v2 到：

`records/K-001端侧推理完整链路.md`

验收标准：

- 文档能够脱离当前聊天独立阅读。
- 能用一条连贯的数据流说明 `模型文件 → Runtime → 原始输入 → 预处理 → Input Tensor → Inference → Output Tensor → 后处理 → 业务结果`。
- 清楚区分一次性初始化工作和重复执行的单次推理工作。
- 回答“需要回答”中的 7 个问题。
- 不把 Runtime、量化、NPU、模型转换等相邻主题偷偷扩展为新的学习目标。

## 后续整合

由 `learning-synthesis` 在母文档 `端侧模型部署到底在做什么.md` 中当前尚未完成的“真正端侧程序如何把这些环节串起来”节点进行整合。Ticket 被 `learning-note` 解决后只标记为 `resolved` 并记录 `result`，不得在解决阶段直接修改母文档或 Learning Map。
