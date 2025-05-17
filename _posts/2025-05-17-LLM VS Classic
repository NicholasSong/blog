---
layout: post
title: "大模型和经典算法的选择"
date: 2025-05-17 10:00:00 +0800
categories: [AI]
tags: [Claude, OpenAi, 大模型, bert, 经典算法]
img:  posts/20250517/llmvsclassic.png
---


# 📘 敏感内容识别实战：从大模型微调到 BERT 微调的探索

在这个项目中，我尝试实现一个多分类文本识别器，目标是判定一段中文文本是否包含敏感或违规内容（如攻击性言论、色情、诈骗、灰色营销等）。  
我对比了两种截然不同的方式来完成这一任务：

1. 使用 Qwen 大模型进行 LoRA 微调（在 Colab 上完成）  
2. 使用 BERT 模型进行监督式训练（在 Paddle AI Studio 中完成）

最终的结果非常明确，在相对固定的场景下：**专业的小模型往往在特定任务中比大模型表现更稳定、成本更低。**
也许在更大更开放的环境综和场景里面，大模型是更好的选择，但是同时也会带来大模型幻觉，以及我们是对模型定义的聪明与否的问题。



## 🧠 第一阶段：Qwen 大模型 + LoRA 微调

我选择了轻量化的大模型 [`Qwen1.5-1.8B-Chat`](https://huggingface.co/Qwen/Qwen1.5-1.8B-Chat)，并在 Google Colab 上使用 LoRA 技术进行了微调。  
使用的训练数据为千级别（~500 条标注数据），数据来自构造的 instruction-style 文本，用于拟合风格分类任务。

### 🛠️ LoRA 微调核心代码框架：

```python
from transformers import AutoTokenizer, AutoModelForCausalLM, Trainer, TrainingArguments
from peft import LoraConfig, get_peft_model, TaskType
from datasets import Dataset

base_model = "Qwen/Qwen1.5-1.8B-Chat"
tokenizer = AutoTokenizer.from_pretrained(base_model, trust_remote_code=True)
model = AutoModelForCausalLM.from_pretrained(base_model, trust_remote_code=True, device_map="auto", torch_dtype=torch.float16)

lora_config = LoraConfig(
    r=8,
    lora_alpha=16,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj"],
    lora_dropout=0.05,
    bias="none",
    task_type=TaskType.CAUSAL_LM
)
model = get_peft_model(model, lora_config)

# ...构造数据集并训练...
```

### ❗遇到的问题总结：

- **训练数据量太小**：千级别的数据难以驱动一个大语言模型微调出有效能力；
- **灾难性遗忘明显**：微调之后模型原有的对话能力严重退化；
- **预测偏差严重**：微调后输出逻辑混乱，模型在结构化判断上不稳定；
- **资源消耗大**：即使是 1.8B 参数模型也对显存有较高要求；
- **训练与推理速度慢**，部署难度高。

### ✅ 收获经验：

我对微调时灾难性遗忘与过拟合非常关注，实际中发现 LoRA 没能很好解决灾难性遗忘的问题，尤其是训练数据量小的情况下反而加剧了模型的结构性损伤。



## ✅ 第二阶段：BERT 中文模型 + PaddleNLP 微调（成功落地）

我使用了 PaddlePaddle 提供的 `bert-base-chinese` 模型，在 AI Studio 上完成了训练。模型结构更轻，训练过程高效稳定。

训练数据量仍为千级别（与 Qwen 一致），标签包括：

- 0 - 正常
- 1 - 攻击性言论
- 2 - 色情
- 3 - 灰色营销
- 4 - 诈骗

### ✅ 训练核心代码（简要）：

```python
from paddlenlp.transformers import BertTokenizer, BertForSequenceClassification

tokenizer = BertTokenizer.from_pretrained("bert-base-chinese")
model = BertForSequenceClassification.from_pretrained("bert-base-chinese", num_classes=5)

# 使用 DataLoader 加载训练数据
# 使用 CrossEntropyLoss、AdamW 优化器、LinearDecayWithWarmup 学习率调度
# 每轮记录 acc/loss，最终保存模型
```

### 📊 训练表现：

- 第 3 个 epoch 后准确率和损失已趋于收敛；
- 在新生成的 100 条真实模拟测试数据上，准确率达到了 **92%**；
- 没有发生过拟合（验证集准确率波动稳定）；
- 模型推理速度快、资源占用低，适合上线部署；
- 类别边界识别效果良好，仅色情 / 攻击性言论有轻微混淆。



## 🧠 我的反思与结论：

| 对比维度 | Qwen + LoRA 微调 | BERT 微调 |
|----------|------------------|------------|
| 模型体积 | 1.8B（大）        | 小（~100M） |
| 所需数据量 | 需要大量对话数据 | 千级别即可 |
| 微调方式 | Causal LM + LoRA | 分类任务监督训练 |
| 训练稳定性 | 差，灾难性遗忘严重 | 高，收敛快 |
| 推理效率 | 慢（显存/成本高） | 快（轻量部署） |
| 部署难度 | 高                | 低 |
| 总结结论 | 不适合小数据分类任务 | 高效实用、极具性价比 |

> **大模型不是通用钥匙，任务明确、样本清晰的小模型反而更有优势。**



## 🔭 后续计划

- ✅ 使用混淆矩阵进一步评估模型各类别表现  
- ✅ 尝试更轻量的中文模型（如 TinyBERT、ERNIE）  
- ✅ 增加边界样本用于数据增强  
- ✅ 使用静态图部署 Paddle 模型为 REST API 服务  



## 🔗 项目进展 / Notebook 持续更新中…

欢迎交流与建议，感谢阅读！

> _「小模型不小看，大模型不盲信。以任务为中心选择方案，才是工程的关键智慧。」_
