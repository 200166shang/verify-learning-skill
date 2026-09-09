---
id: K-001
type: concept
status: open
map_node: end-to-end-flow
---

# KnowledgeTicket: 端侧程序如何把模型部署各环节串起来？

## Question

一个训练完成的模型进入真实端侧程序后，模型文件、Runtime、输入数据、预处理、Input Tensor、Forward/Inference、Output Tensor、后处理和业务结果之间，究竟按照怎样的执行链路连接起来？

## Why this matters

这是当前 Learning Map 中唯一尚未完成的正式节点。解决它的目的不是扩展新的端侧部署知识树，而是把已经学习过的 Tensor、Forward、训练与推理、模型文件、预处理和后处理第一次串成一条完整部署主线。

## Scope

- 解释一条典型端侧推理调用链。
- 明确每个阶段的输入、输出以及职责边界。
- 说明 Runtime 在链路中的位置，但不在本 Ticket 中展开具体 Runtime、模型格式转换、量化、NPU 或 Delegate 的内部实现。
- 以能够阅读真实端侧部署模块的主流程为理解标准。

## Expected result

产出一个可独立阅读的 LearningRecord，用一条连贯执行链解释端侧推理主流程，并指出后续真正需要继续追问的缺口（如有）。
