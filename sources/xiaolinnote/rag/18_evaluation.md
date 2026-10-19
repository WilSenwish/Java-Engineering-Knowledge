# RAG 面试题 18：怎么量化你的 RAG 效果？

> **来源站点**：小林面试笔记（xiaolinnote.com）
> **原文链接**：https://xiaolinnote.com/ai/rag/18_evaluation.html
> **抓取日期**：2026-10-20
> **所属知识域**：域6 性能优化与架构演进（AI 前沿）
> **板块**：RAG 面试题（23 题）· 工程实践

---

## 核心答案

两层评估。**检索层**：Hit@K（找到没）+ MRR（多快找到的）。**生成层**：RAGAs 框架 LLM-as-a-Judge——Faithfulness（有没有幻觉）/ Answer Relevancy（答没答对所问）/ Context Recall（检索覆盖够不够）/ Context Precision（有用的排没排前面）。**必须在自己的业务数据上跑，通用排行榜不代表你的场景**。线上指标（踩率/追问率/转人工率）才是最终衡量标准，离线只是辅助。闭环：离线测评→上线→线上观测→发现问题→离线复现→修复→再上线。

## 检索层指标

准备「问题 + 正确 chunk ID」测评集（历史问答标注或专家整理）。

- **Hit@K**：前 K 条里有没有正确 chunk——Hit@5 = 0.9 即 90% 问题前 5 条能找到。**Hit@5 < 0.7 检索层有问题**（换 Embedding 或优化 Chunking）；> 0.8 检索 OK，答案还差问题在生成层；
- **MRR**（Mean Reciprocal Rank）：`1/排名` 求平均——第一得 1 分第二得 0.5 第五得 0.2。**MRR < 0.5 通常 Rerank 不够好**（召回了但没排前面）。

易混淆点：Hit@K 是「找到没」，MRR 是「多快找到的」——Hit@5=0.9 但 MRR=0.3 说明内容找到了但排第 4、5 才出现。配合起来精确定位检索层问题。

## 生成层指标（RAGAs）

LLM-as-a-Judge 自动打分免人工标注。成本缓解：核心测试集抽样 200~500 条；裁判模型 GPT-4o 降级 GPT-4o-mini（成本降 10 倍精度损失可接受）。

- **Faithfulness（忠实度）**：答案每句话在 chunk 里有没有出处——衡量幻觉，目标 > 0.8；
- **Answer Relevancy**：答案有没有回答用户问的——**和 Faithfulness 两回事**：问「北京天气」答北京历史全对但不相关 = Faithfulness 高 Relevancy 低；
- **Context Recall**：回答问题所需信息有多少比例被检索覆盖（需 ground_truth）；
- **Context Precision**：召回的 10 个里相关那几个排没排前面（需 ground_truth）。

## 指标定位问题

| 指标低 | 问题 | 优化方向 |
|--------|------|---------|
| Context Recall | 检索漏了必要信息 | 换 Embedding/调 Chunking/多路召回 |
| Context Precision | 召回太多噪音稀释注意力 | 加强 Rerank/减少送 LLM 的 chunk 数 |
| Faithfulness | LLM 编造 | Prompt 约束/引用核查/质量门控 |
| Answer Relevancy | 跑题 | Prompt 明确「严格回答问题本身」 |

## 线上指标（最终验收）

踩率（最直接负反馈）/ 追问率（答非所问程度）/ 转人工率（知识库覆盖不足；但质量门控致其上升不一定是坏事——宁可转人工不给错答案）/ 空回答率（主动说不知道的比例，过高=知识库该扩）/ 会话解决率（最综合最贴近体验）。**离线好不代表线上好**——离线集不代表真实分布，或指标优化过头损害体验（为 Faithfulness 把 Prompt 收太死，模型过于保守用户嫌不好用），两者定期交叉对照。
