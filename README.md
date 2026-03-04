# 🏆 CAIB-500: 中文招投标开源数据集

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Data Count](https://img.shields.io/badge/Total_Cases-532-blue)]()
[![Collusion Cases](https://img.shields.io/badge/Collusion_Cases-103-red)]()
[![Features](https://img.shields.io/badge/Feature_Dim-12-orange)]()

> **首个包含细粒度围串标特征向量（12-Dim）的中文招投标开源数据集**

## 📖 项目简介

**CAIB-500** (Chinese AI Bidding O彭Dataset) 是一个专注于**招投标围串标识别和招标文件检测**的高质量开源数据集。本数据集基于公开的政府采购真实行政处罚决定书构建，包含 **532条** 经过深度清洗、脱敏和结构化标注的案例。

与传统文本数据集不同，CAIB-500 创新性地引入了**层次化标注体系**，特别是针对高价值的“围标串标”场景，提取了**12维细粒度特征向量**（涵盖技术指纹、内容相似度、行为模式等），为训练高精度的招投标违规检测模型、处罚预测系统及招投标大模型提供了稀缺的“黄金标准”数据。

## ✨ 核心亮点与创新

### 1. 独创的 12 维围串标特征向量 (Core Innovation)
我们不仅仅标注了“是否违规”，更深入解析了“如何违规”。针对 **103 条 (19.4%)** 围串标案例，我们定义了 6 大模块、12 个具体特征维度：

| 特征类别 | 特征代码 | 含义说明 | 出现频次 (Top) |
| :--- | :--- | :--- | :--- |
| **🖥️ 技术指纹** | `IP_ADDRESS_SAME` | 不同投标人 IP 地址一致 | 12 |
| | `MAC_ADDRESS_SAME` | 网卡 MAC 地址一致 | - |
| | `HARDWARE_FINGERPRINT` | 硬盘序列号/CPU ID 一致 | - |
| **📄 内容特征** | `FILE_CONTENT_SAME` | **投标文件内容非正常一致** | **55 (最高频)** |
| | `QUOTE_PATTERN` | 报价呈规律性差异/相同 | 8 |
| | `TYPO_PATTERN` | 相同的特殊文字错误/乱码 | 6 |
| **👤 行为特征** | `UPLOAD_PATTERN` | 上传时间/操作习惯高度重合 | 16 |
| | `CONTACT_INFO_SAME` | 联系人/电话/邮箱相同 | 4 |
| | `LEGAL_REP_SAME` | 法定代表人相同 | 2 |
| **📜 资质特征** | `AUTHORIZATION_FORGED` | 授权书伪造或异常 | 14 |
| | `QUALIFICATION_SAME` | 资质材料混用/雷同 | - |
| | `CREATOR_SAME` | 文档创建者/最后修改者相同 | - |

### 2. 多维度的场景分类
数据覆盖 5 大核心违规场景，分布清晰：
*   🔴 **Other_Violation (57.0%)**: 履约评价差、未按时进场等其他违规。
*   🔵 **False_Material (23.7%)**: 提供虚假材料、伪造证书、假发票等。
*   🟢 **Collusion (19.4%)**: 围标、串通投标（含上述 12 维特征）。
*   🟠 **Bribery**: 涉及行贿犯罪的关联记录。
*   🟣 **Malicious_Complaint**: 恶意投诉与捏造事实。

### 3. 完整的处罚结构化信息
每条数据均包含结构化的处罚结果，支持回归分析与量刑预测研究：
*   **罚款金额分布**: 中位数约 **10,472 元**，长尾分布明显（最高达数十万）。
*   **禁止期限**: 明确标注禁止参与政府采购的年限（常见为 1-3 年）。
*   **法规引用**: 自动匹配并人工校验了适用的法律条款（如《深圳经济特区政府采购条例》）。

## 📊 数据统计概览

![数据统计看板](images/stats_dashboard.png)

*   **特征共现分析**: 热力图显示 `FILE_CONTENT_SAME` 常与 `UPLOAD_PATTERN` 和 `IP_ADDRESS_SAME` 高度共现，揭示了典型的投标违规模式。
*   **数据集划分**: 未划分数据集，后续补充数据量到一定规模后划分，或自行划分。

## 🚀 模型应用场景映射

本数据集专为以下 AI 任务设计：

| 应用场景 | 推荐模型类型 | 输入 (Input) | 输出 (Output) | 对应数据字段 |
| :--- | :--- | :--- | :--- | :--- |
| **🕵️ 围串标识别** | 多标签分类 / GNN | 多份投标文件文本/元数据 | 风险评分 + **12维特征向量** | `raw_text` → `collusion_features` |
| **🔍 虚假材料检测** | NER / 文本分类 | 单份投标文件 | 虚假点位定位 + 风险类型 | `text_content` → `violation_analysis` |
| **⚖️ 智能处罚预测** | 回归模型 / LLM | 违规事实描述 + 特征向量 | 预测罚款金额 + 禁止期限 | `features` → `penalty_info` |
| **📚 法规推荐引擎** | RAG / 语义匹配 | 违规案情摘要 | 推荐适用法条及依据 | `violation_behavior` → `legal_reference` |
| **🔗 相似案例检索** | Vector Search | 新案件描述 | Top-K 历史相似判例 | `text_embedding` |

## 📂 数据格式说明

数据以 `JSONL` 格式存储，每条样本结构如下：

```json
{
  "case_id": "CAIB-GD-SZ-LH-0112",
  "scene_classification": ["Collusion", "False_Material"],
  "collusion_features": {
    "IP_ADDRESS_SAME": true,
    "FILE_CONTENT_SAME": true,
    "UPLOAD_PATTERN": true,
    "...": false
  },
  "penalty_information": {
    "fine_amount": 15000,
    "ban_duration_years": 2,
    "penalty_type": ["Fine", "Ban"]
  },
  "legal_reference": [
    {"law": "深圳经济特区政府采购条例", "article": "第五十七条"}
  ],
  "text_content": {
    "raw_text_anonymized": "供应商A与供应商B的投标文件由同一台电脑上传...",
    "summary": "两家公司投标文件雷同且IP一致"
  }
}
## 数据集更新
预计每两周更新一次数据集，更新主要内容包括增加数据量，更新数据结构，发布预训练的小型招投标语言模型，提供效果展示网页等。
