# prepare_data 目录说明

本目录包含用于**原始医疗数据采集与预处理**的三个脚本，是整个知识图谱问答系统的数据准备阶段。各文件职责如下：

---

## 1. `data_spider.py` —— 医疗数据爬虫

**功能：** 从[寻医问药网](http://jib.xywy.com)批量爬取疾病相关信息，并将原始数据存入本地 MongoDB 数据库。

**主要工作：**

- 遍历寻医问药网约 11,000 个疾病页面，针对每种疾病分别请求以下 8 类子页面：
  | 子页面类型 | URL 模板 |
  |---|---|
  | 疾病概述（基本信息） | `.../gaishu/{id}.htm` |
  | 病因 | `.../cause/{id}.htm` |
  | 预防 | `.../prevent/{id}.htm` |
  | 症状 | `.../symptom/{id}.htm` |
  | 检查项目 | `.../inspect/{id}.htm` |
  | 治疗 | `.../treat/{id}.htm` |
  | 饮食宜忌 | `.../food/{id}.htm` |
  | 相关药品 | `.../drug/{id}.htm` |

- 使用 `lxml` 对 HTML 进行 XPath 解析，提取疾病名称、分类、简介、属性、症状、治疗方案、饮食建议、推荐药品等结构化字段。
- 额外爬取约 3,685 个**检查项目**页面（`http://jck.xywy.com/jc_{id}.html`），将 HTML 原文存入 MongoDB 的 `jc` 集合，供后续解析检查项名称使用。
- 所有数据写入本地 MongoDB 的 `medical` 数据库、`data` 集合。

---

## 2. `build_data.py` —— 数据清洗与结构化转换

**功能：** 读取 MongoDB 中由爬虫采集的原始数据，进行清洗、字段映射和结构化处理，将结果写入 MongoDB 的 `medical` 集合，作为后续知识图谱构建的输入。

**主要工作：**

- 从 `data` 集合中逐条读取爬虫存储的原始文档。
- 清洗文本字段（去除多余的换行、制表符、空白字符等）。
- 将中文字段名统一映射为英文字段名（如 `"就诊科室"` → `cure_department`，`"症状"` → `symptom` 等），便于程序统一处理。
- 对特定字段进行结构化处理：
  - `cure_department`、`cure_way`、`common_drug`：按空格分割为列表。
  - `acompany`（并发症）：调用 `max_cut.py` 中的双向最大匹配分词算法切分词语，过滤单字词。
  - 其他字符串字段：去除多余空白与制表符。
- 解析检查项目 URL，查询 `jc` 集合获取对应检查项名称。
- 将处理后的结构化数据插入 `medical` 集合，供 `build_medicalgraph.py` 读取构建知识图谱。

---

## 3. `max_cut.py` —— 中文分词（双向最大匹配）

**功能：** 实现基于词典的**双向最大匹配（Bi-directional Maximum Matching）**中文分词算法，用于对并发症等文本字段进行分词。

**主要工作：**

- 从 `disease.txt` 词典文件加载已知词汇，记录最长词长度。
- **正向最大匹配（`max_forward_cut`）：** 从左到右，每次尽可能取最长词进行匹配；未命中则按单字切分。
- **逆向最大匹配（`max_backward_cut`）：** 从右到左，每次尽可能取最长词进行匹配；未命中则按单字切分。
- **双向最大匹配（`max_biward_cut`）：** 综合正向与逆向结果，按启发式规则选取最优切分：
  1. 若两者词数不同，取词数较少的结果。
  2. 若词数相同且结果不同，取单字词较少的结果（歧义更少）。

---

## 数据流向总结

```
寻医问药网 (xywy.com)
      │
      ▼
data_spider.py  ── 爬取原始 HTML，解析后存入 MongoDB (medical.data / medical.jc)
      │
      ▼
build_data.py   ── 清洗、字段映射、分词（调用 max_cut.py），写入 MongoDB (medical.medical)
      │
      ▼
build_medicalgraph.py  ── 读取结构化数据，构建 Neo4j 知识图谱
```
