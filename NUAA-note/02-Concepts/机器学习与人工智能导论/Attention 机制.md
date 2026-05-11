---
type: concept
course: 机器学习与人工智能导论
chapter: [3]
first_seen: 第4讲
prerequisites: [LSTM（长短期记忆网络）, 循环神经网络（RNN）, 神经网络]
related: [Transformer, LSTM（长短期记忆网络）, 大语言模型（LLM）]
contrasts: [LSTM（长短期记忆网络）, 循环神经网络（RNN）]
---

# Attention 机制

## 定义

Attention（注意力机制）是一种让模型在处理序列数据时**动态聚焦于输入中最重要的部分**的计算机制。核心思想是：对于输出序列的每一步，模型根据 Query 与所有输入位置的 Key 计算相似度（注意力权重），再对 Value 进行加权聚合。其标志性公式为：

$$
\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$

## 直觉理解

好比你在嘈杂的派对上听朋友说话——周围所有人都在同时说话（所有输入信息），但你选择性地"调高"朋友声音的音量（高注意力权重）、"调低"其他噪音（低注意力权重）。Attention 机制就是让计算机学会这种"选择性关注"的能力。

## 前置概念

- 编码器-解码器（Seq2Seq）：传统 Seq2Seq 模型用固定长度向量表示输入序列，Attention 允许动态聚焦
- QKV（Query-Key-Value）：Query 问"我要找什么"，Key 说"我有什么"，Value 是"实际内容"
- 点积相似度：Q 与 K 的内积衡量两者的匹配程度

## 推导到 / 关联到

- 自注意力（Self-Attention）：Q=K=V 都来自同一序列，捕捉序列内部的长程依赖
- 多头注意力（Multi-Head Attention）：多组 QKV 并行计算，捕捉不同层面的关联
- Transformer：完全基于 Attention（抛弃 RNN）的架构，是大语言模型的基础
- 位置编码（Positional Encoding）：Attention 无时序概念，需额外编码位置信息

## 易混概念

- Attention vs LSTM：LSTM 通过门控按顺序逐步传递信息；Attention 直接计算任意两个位置之间的关系，并行度高
- 软注意力 vs 硬注意力：软注意力可微（权重连续，可用反向传播训练）；硬注意力是离散采样（需强化学习训练）
- Self-Attention vs Cross-Attention：Self-Attention 在同一序列内部计算；Cross-Attention 在不同序列之间计算（如编码器-解码器）

## 典型例子

- 机器翻译：编码"我在吃饭"→ 解码输出 "I am eating" 时，"I" 高权重关注 "我"，"eating" 高权重关注 "吃"
- Transformer 架构：完全依靠多头自注意力 + 前馈网络，无任何循环结构，成为 GPT/BERT 等大模型的基础
- 图像 Attention：看图描述任务中，生成"狗"这个词时，注意力权重集中在图像中狗的区域
