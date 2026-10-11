# TODO

[ROADMAP.md](./ROADMAP.md) 的执行清单。每条给出可跑判据，判据通过才勾选；`[ ]` 未做，`[x]` 已做。

**数据源**：本地 `~/repos/quanttide/domains/`，全程离线，不依赖 GitHub。

## 阶段 0 向量化（三件共同的闸门）

- [ ] **0.1 片段提取**——读本地语料，切片段：清洗模板文件（`README.md`、`CHANGELOG.md`、`.agents/`、`.quanttide/`）、按内容哈希去重、剔除纯标题/代码块/表格，保留 20 至 900 字且含 ≥15 汉字的段落。
      规则参考 [`reference/build_eval.py`](./reference/build_eval.py) 的 `clean_block` / `extract` / `collect`——旧采样脚本的提取逻辑，网络与评测集部分已废弃，只借这一段。
      产出：`data/fragments.tsv`，三列 `id, text, source`，`source` 形如 `领域仓#相对路径`。
      判据：条数与既有统计同量级（约 2.4 万条）；断网可跑；抽 10 条人工看是否成句。
- [ ] **0.2 选 embedding 模型**——本地 ollama 拉一个向量模型（`all-minilm` 约 46MB，或 `nomic-embed-text` 约 274MB），脚本头部写死模型名与向量维度。
      判据：`ollama list` 有该模型；一条 50 字中文文本能拿到向量，维度与声明一致。
      说明：显存 3GB，向量模型都很小，无压力。
- [ ] **0.3 向量化落盘**——批量算向量并落盘（`data/embeddings.npy` + `data/embedding_ids.tsv` 对照）。
      判据：向量行数 == `fragments.tsv` 行数；失败条目如实报条数，不静默丢。

## 阶段 1 找重复

- [ ] **1.1 算高分对**——算片段两两相似度（或近邻检索）取高分对；阈值与算法先写死，跑完不改，要改另起一轮并说明。
      产出：`data/duplicates.tsv`，五列 `id_a, id_b, score, source_a, source_b`。
      判据：抽 30 对人工看，记录误报数；报告里写清阈值、取样量与分母。
- [ ] **1.2 出报告**——`docs/duplicates.md`。
      判据：结论先行、术语解释、数字给分母、附复跑命令；把「跨领域重复」与「同领域内重复」分开列。

## 阶段 2 找关联

- [ ] **2.1 聚类**——对向量聚类，算法与参数先写死。
      产出：`data/clusters.tsv`（`cluster_id, id, source`）+ 每簇主题摘要。
      判据：簇覆盖率（被聚进簇的片段占比）写清；抽几个簇人工判读有没有意义。
- [ ] **2.2 出报告**——`docs/clusters.md`，贴出抽检簇的成员供人工判读。

## 阶段 3 建索引

- [ ] **3.1 建索引与查询入口**——`scripts/search.py "<主题>"`，返回最相近的片段与出处。
- [ ] **3.2 真实主题实测**——拿几个你心里真实的主题查，如实记录命中与不命中。
