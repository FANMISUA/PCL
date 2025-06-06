<!--

# Daily Report（yyyy-mm-dd）

## 🎯 任务

## 🔧 具体内容

## 📝 明日计划

## 💡 思考
-->

# Daily Report（2025-05-30）

## 🎯 任务

- [ ] 修改 HJ 人才工作计划： 以个人为主体，个人工作经历能给项目带来什么。
- [ ] 文献： 多模态； 长文本；
- [ ] 配置云脑 2 环境。

## 🔧 具体内容

### 文献 Med-Gemini [arxiv.org/abs/2404.18416](https://arxiv.org/abs/2404.18416)

### Med-Gemini: Capabilities of Gemini Models in Medicine

#### 📝 文章总结

Med-Gemini 是基于 Gemini 架构开发的多模态大型语言模型（LLMs），专为医学领域设计，具备多模态输入（文本、图像、视频）和网页搜索功能。论文在 14 个医学基准测试中，10 个任务刷新了 SOTA 纪录，优于 GPT-4 系列。特别是在 MedQA（USMLE）测试中，最佳模型准确率达到 91.1%，并在 NEJM 图像挑战等多模态任务中有显著优势。

#### ⚙️ 具体方法

- **模型架构**：采用 Gemini 架构，集成了网页搜索功能和自定义编码器，支持多模态输入。
- **训练与评估**：
  - 使用多模态医学数据集进行微调，包括医学文本、图像和视频。
  - 引入不确定性引导搜索策略（uncertainty-guided search）提升推理准确率。
  - 长上下文推理：在医疗视频问答和超长文本任务（例如医疗记录）中，通过上下文学习完成端到端推理。
- **基准测试**：涵盖 MedQA（USMLE）、NEJM 图像挑战、MMMU（健康与医学）等 14 个任务。

#### ⚠️ 存在问题

- 真实医疗场景中尚未经过严格的安全性与有效性评估。
- 部分多模态任务依赖上下文提示，对实际临床部署仍需进一步验证。
- 论文未提供详细的模型参数和推理机制细节，外部复现和验证较难。

# Daily Report（2025-06-06）

## 🎯 基因会议

### 华大知识图谱

- **服务器**：
  - 192.168.237.73
  - 用法：[Condor 集群使用指南](Condor 集群使用指南.md)
- /mnt/net1/jrchen/1KG/ref_gatk
  - preporcess_bert.py
  - preprocess_data.py
- /mnt/net1/jrchen/ECOLE_reproduction/ECOLE_Training/

### 病原数据检测
