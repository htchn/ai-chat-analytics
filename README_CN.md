[📖 English Version](README.md)

# 🤖 AI Chat Analytics

> 一套完整的对话式AI产品用户行为分析流水线，从原始事件数据到可执行的产品优化建议。

## 🎯 这是什么？

如果你做了一个类似 ChatGPT/Claude 的产品，用户会聊天、点赞 👍、点踩 👎、重新生成 🔄、追问 💬 —— 或者什么都不做就走了 🤐。**怎么把这些信号变成产品决策？**

本项目展示了一套完整的分析框架：

- 📊 **整体表现如何？** — 行为分布、满意度指标
- 🔍 **哪里有问题？** — 按意图场景拆解质量指标
- ❓ **为什么不满意？** — 语义归因分析，定位根因
- 💬 **追问到底是好事还是坏事？** — 探索式 vs 纠错式信号拆解
- 🎯 **应该优先修什么？** — 优先级排序 + 可执行建议

---

## 🧠 分析思维链与思维树

### 总览

```mermaid
graph TD
    Q0{"🤔 产品有问题吗？"}
    Q0 --> M["📊 02 行为指标<br/>怎么衡量？"]
    M --> Q1{"哪里有问题？"}
    Q1 --> S["🔍 02 场景拆解<br/>哪个意图最差？"]
    S --> Q2{"为什么不满意？"}
    Q2 --> A["🔬 03 语义归因<br/>根因分析"]
    Q2 --> F["💬 04 追问拆解<br/>隐藏信号挖掘"]
    A --> Q3{"应该先修什么？"}
    F --> Q3
    Q3 --> R["🎯 05 优先级矩阵<br/>+ 监控体系"]

    style Q0 fill:#6366f1,stroke:#4f46e5,color:#fff
    style Q1 fill:#f97316,stroke:#ea580c,color:#fff
    style Q2 fill:#e74c3c,stroke:#dc2626,color:#fff
    style Q3 fill:#2ecc71,stroke:#16a34a,color:#fff
    style M fill:#ede9fe,stroke:#6366f1,color:#333
    style S fill:#fff7ed,stroke:#f97316,color:#333
    style A fill:#fee2e2,stroke:#e74c3c,color:#333
    style F fill:#dbeafe,stroke:#3498db,color:#333
    style R fill:#dcfce7,stroke:#2ecc71,color:#333
```

### 分支一 — "产品有问题吗？" → 行为指标体系

```mermaid
graph TD
    ROOT["📊 怎么衡量产品质量？"]
    ROOT --> EXPLICIT["显性信号"]
    ROOT --> IMPLICIT["隐性信号"]
    ROOT --> TIME["时间维度"]
    ROOT --> USER["用户维度"]

    EXPLICIT --> LIKE["👍 点赞率<br/>频率统计 / 总对话数"]
    EXPLICIT --> DISLIKE["👎 点踩率<br/>频率统计 / 总对话数"]
    EXPLICIT --> RETRY["🔄 重新生成率<br/>隐性差评——用户不满意<br/>但没有明确点踩"]
    EXPLICIT --> FOLLOWUP["💬 追问率<br/>模糊信号——好坏未知<br/>→ 04 中拆解"]

    IMPLICIT --> SILENT["🤫 沉默率<br/>方法：总对话数 − 有事件的对话数<br/>为什么不能忽略？沉默用户不在事件表中<br/>直接统计事件会高估满意度"]

    TIME --> DAILY["日级趋势<br/>日聚合 + 多指标叠加时序图<br/>作用：发现模型更新导致的突变"]

    USER --> ACTIVE["活跃天数分布<br/>按用户类型分组 + 箱线图<br/>重度 / 轻度 / 流失"]
    USER --> RETENTION["留存曲线<br/>日级留存率 + eligible用户过滤<br/>排除观察期外用户避免幸存者偏差"]

    ROOT --> FINDING["📋 发现：<br/>24%沉默，14%点踩，19%重试<br/>点赞率 ≈ 沉默率 →<br/>满意的用户也不一定会反馈"]

    style ROOT fill:#6366f1,stroke:#4f46e5,color:#fff
    style FINDING fill:#fef3c7,stroke:#f39c12,color:#333
```

### 分支二 — "哪里有问题？" → 场景拆解

```mermaid
graph TD
    ROOT["🔍 哪个场景问题最多？"]

    ROOT --> PREREQ["前提：意图分类"]
    PREREQ --> IC1["本项目：预分类字段<br/>模拟分类模型输出"]
    PREREQ --> IC2["生产环境四种方式<br/>关键词规则 / LLM / BERT微调 / 人工标注"]

    ROOT --> METHOD["怎么拆？"]
    METHOD --> CROSS["交叉列联表<br/>pd.crosstab intent × event_type<br/>用 normalize='index'<br/>因为各场景对话量不同——看比例不看绝对数"]
    METHOD --> RANK["怎么排序？<br/>不满意率 = 点踩率 + 重试率<br/>为什么合并？retry 本质是隐性差评"]

    ROOT --> FINDING["📋 发现"]
    FINDING --> F1["🔴 代码生成：45.1%<br/>retry率30%拉高不满意率"]
    FINDING --> F2["🔴 知识问答：39.3%<br/>dislike率24%最高"]
    FINDING --> F3["🟡 数据分析：39.4%"]
    FINDING --> F4["🟡 创意写作：25.7%"]
    FINDING --> F5["🟢 翻译：19.8%"]
    FINDING --> F6["🟢 闲聊：9.7%<br/>65%沉默但属于场景特性"]

    style ROOT fill:#f97316,stroke:#ea580c,color:#fff
    style FINDING fill:#fef3c7,stroke:#f39c12,color:#333
```

### 分支三 — "为什么不满意？" → 语义归因

```mermaid
graph TD
    ROOT["🔬 对7,363条bad case<br/>做根因分析"]

    ROOT --> TAXONOMY["归因体系 — 5个类别"]
    TAXONOMY --> T1["🟥 幻觉<br/>编造事实 / 人物 / 论文"]
    TAXONOMY --> T2["🟧 答非所问<br/>理解偏差，回答方向偏离"]
    TAXONOMY --> T3["🟪 拒绝回答<br/>过度安全策略，该答不答"]
    TAXONOMY --> T4["🟩 格式不匹配<br/>要代码给了文字说明"]
    TAXONOMY --> T5["📘 质量不足<br/>方向对但太笼统——兜底类"]

    ROOT --> PATHA["路径A：关键词规则<br/>+ 人工校验"]
    PATHA --> PA1["第一步：优先级规则链<br/>拒答关键词 → 幻觉信号<br/>→ 格式检查 → 答非所问信号<br/>→ 兜底：质量不足"]
    PATHA --> PA2["第二步：人工校验<br/>导出csv → 标注员检查规则结果<br/>重点：被兜底的case是否有更具体归因<br/>质量控制：双人标注 + Cohen's Kappa"]
    PATHA --> PA3["局限：71%兜底到质量不足<br/>翻译和闲聊100%兜底"]

    ROOT --> PATHB["路径B：LLM自动打标"]
    PATHB --> PB1["方法：zero-shot prompt分类<br/>输入：query + response → prompt模板<br/>→ 返回归因类别"]
    PATHB --> PB2["模型：gpt-4o-mini<br/>temperature=0 确保结果可复现"]
    PATHB --> PB3["校验：用路径A的人工标注<br/>作为golden set评估准确率"]

    ROOT --> FINDING["📋 发现"]
    FINDING --> F1["知识问答 → 幻觉22%"]
    FINDING --> F2["数据分析 → 格式不匹配65%"]
    FINDING --> F3["创意写作 → 拒绝回答23%"]
    FINDING --> F4["代码生成 → 拒答12% + 格式13%"]

    style ROOT fill:#e74c3c,stroke:#dc2626,color:#fff
    style FINDING fill:#fef3c7,stroke:#f39c12,color:#333
```

### 分支四 — "追问是好事还是坏事？" → 信号拆解

```mermaid
graph TD
    ROOT["💬 追问率19.4%<br/>这是好事还是坏事？"]

    ROOT --> WHY["为什么要拆？<br/>追问是模糊信号——<br/>可能是好奇，也可能是不满"]

    ROOT --> HOW["怎么分？"]
    HOW --> CORRECTIVE["🔴 纠错式<br/>关键词检测：'不是' '你理解错'<br/>'有问题' '不对' '你没有回答'"]
    HOW --> EXPLORATORY["🟢 探索式<br/>默认：不含纠错关键词"]
    HOW --> PROD["生产环境改进：<br/>用LLM做情感/意图分类<br/>替代关键词规则"]

    ROOT --> SCENE["场景关联<br/>交叉分析：intent × followup_type"]
    ROOT --> BEHAVIOR["行为关联<br/>时序事件链：<br/>按时间戳排序 → 取追问之后的首个事件"]

    ROOT --> FINDING["📋 发现"]
    FINDING --> F1["知识问答纠错率64% 🚨<br/>与03的幻觉问题高度吻合"]
    FINDING --> F2["创意写作纠错率仅10%<br/>追问多为正向探索"]
    FINDING --> F3["洞察：纠错式追问可作为<br/>自动化质量监控的信号源"]

    style ROOT fill:#3498db,stroke:#2563eb,color:#fff
    style FINDING fill:#fef3c7,stroke:#f39c12,color:#333
```

### 分支五 — "应该先修什么？" → 优先级矩阵

```mermaid
graph TD
    ROOT["🎯 优先级排序"]
    ROOT --> LOGIC["排序逻辑：<br/>不满意率 × 纠错追问率<br/>× 归因严重度"]

    ROOT --> P0["🔴 P0 — 立即优化"]
    P0 --> P0A["知识问答<br/>根因：幻觉<br/>措施：接入RAG + 置信度检测<br/>低置信时标注'我不确定'"]
    P0 --> P0B["代码生成<br/>根因：拒答 + 低质量<br/>措施：代码沙箱 + prompt策略优化<br/>直接给代码，不要只给思路"]

    ROOT --> P1["🟡 P1 — 短期优化"]
    P1 --> P1A["数据分析<br/>根因：格式不匹配<br/>措施：默认输出代码块结构化回答<br/>先给方案再问需求"]
    P1 --> P1B["创意写作<br/>根因：过度安全<br/>措施：放宽创意类安全规则<br/>风格模仿 ≠ 版权侵犯"]

    ROOT --> P2["🟢 P2 — 持续监控"]
    P2 --> P2A["翻译 — 表现稳定，保持监控"]
    P2 --> P2B["闲聊 — 高沉默属场景特性"]

    ROOT --> MONITOR["📡 监控体系"]
    MONITOR --> MON1["每日：DAU、留存率、<br/>点踩率、重试率"]
    MONITOR --> MON2["每周：各场景不满意率、<br/>归因分布、纠错追问率"]
    MONITOR --> MON3["告警：点踩率日环比>5%<br/>幻觉占比>15%<br/>任一场景不满意率>50%"]

    style ROOT fill:#2ecc71,stroke:#16a34a,color:#fff
    style P0 fill:#fee2e2,stroke:#e74c3c,color:#333
    style P1 fill:#fef3c7,stroke:#f39c12,color:#333
    style P2 fill:#dcfce7,stroke:#2ecc71,color:#333
```

---

## 📁 项目结构

```
ai-chat-analytics/
├── README.md                          # English (default)
├── README_CN.md                       # 中文版
├── requirements.txt
├── data/
│   ├── users.csv                      # 500 名模拟用户
│   ├── conversations.csv              # ~22K 条对话
│   └── events.csv                     # ~20K 条行为事件
├── notebooks/
│   ├── 01_data_generation.ipynb       # 合成数据生成
│   ├── 02_behavior_analysis.ipynb     # 多维行为指标分析
│   ├── 03_semantic_attribution.ipynb  # Bad case 归因分析
│   ├── 04_followup_signal.ipynb       # 追问信号拆解
│   └── 05_insights_summary.ipynb      # 洞察汇总与产品建议
├── src/
│   └── utils.py                       # 公共工具函数
└── output/
    └── bad_cases_for_labeling.csv     # 导出供人工标注
```

## ⚡ 快速开始

```bash
# 克隆仓库
git clone https://github.com/gh59/ai-chat-analytics.git
cd ai-chat-analytics

# 安装依赖
pip install -r requirements.txt

# 按顺序运行 notebook
cd notebooks
jupyter notebook 01_data_generation.ipynb
```

## 📦 依赖

```
pandas
numpy
plotly
jupyter
```

## 🧩 设计理念

1. **不预打标签** — 意图分类和归因标签不内置在合成数据中，流水线展示的是如何从原始数据中推导，贴近真实场景。

2. **事件流模型** — 一条对话可以触发多个事件（先重试再点赞），而不是简单的一对一标签。

3. **沉默即数据** — 24%的对话没有任何反馈。忽略它们会让你的质量指标虚高。

4. **追问是模糊信号** — 高追问率可能意味着高参与度，也可能意味着高挫败感。必须拆分后才能解读。

5. **归因需要多条路径** — 关键词规则快但浅，LLM打标准但贵，人工标注精但慢。生产系统三者缺一不可。
