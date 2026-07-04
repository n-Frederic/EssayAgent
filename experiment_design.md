# 实验设计记录

## 项目定位

我们目前希望构建一个“代码复现 + 自动优化”的 agent：它能够围绕一篇目标论文或一个目标算法模块，完成代码复现、实验运行、结果分析，并在可验证反馈的基础上自动提出和执行优化方案。

## 当前目标论文

本轮实验以 NatureBench PDF 中涉及 MIRO 的源论文作为主目标：

- 论文：Pineda et al., "Enhanced spatial clustering of single-molecule localizations with graph neural networks"
- 原论文 PDF：https://arxiv.org/pdf/2412.00173
- 来源：Nature Communications, 16(1):9693, 2025
- 方法：MIRO (Multifunctional Integration through Relational Optimization)
- 任务：对单分子定位显微镜（SMLM）产生的 localization point cloud 进行空间聚类增强；MIRO 使用 recurrent graph neural networks 对点云进行变换，使 DBSCAN 等传统聚类方法在复杂形状、多尺度、高噪声或高密度场景下更接近 ground truth。

## 论文选择要求与 comment

目标论文应满足以下条件：

1. 来源为 CNS 或其子刊级别论文。
2. 方法与实验流程相对清晰，具备较好的代码复现可行性。
3. 存在较多可分析、可验证、可优化的改进空间。

Comment:

- 级别：满足。NatureBench 的参考文献中将 MIRO 源论文列为 Nature Communications 2025 论文，符合 CNS 子刊级别要求。
- 易复现：较适合。任务输入是 SMLM 点云，核心流程可拆为数据构造、图构建、recurrent GNN 点云变换、DBSCAN 聚类、指标评估；相较大规模 foundation model，复现闭环更轻。
- 易优化：较适合。可优化点集中在图构建、GNN 结构、训练目标、数据增强、聚类后处理和超参数搜索，且都能通过聚类指标形成自动反馈。

## Agent 复现目标

Agent 需要围绕 MIRO 完成以下复现闭环：

1. 读取论文、代码和数据说明，提取最小可运行实验配置。
2. 构建 baseline：直接对 SMLM localization point cloud 运行 DBSCAN，并记录与 ground truth 的差距。
3. 复现 MIRO：实现或调用 recurrent graph neural network，将原始点云变换为更易聚类的表示，再接 DBSCAN 得到聚类结果。
4. 建立统一评估脚本：至少记录 ARI、NMI/AMI、precision、recall、F1，以及簇数量误差等可自动比较指标。
5. 固化实验记录：每次运行保存配置、随机种子、指标、图像可视化和失败原因，方便 agent 后续根据反馈优化。

## Agent 自动优化方向

优化不应只停留在调参，而应围绕可验证反馈形成小步实验：

1. 图构建优化：比较 kNN、radius graph、混合图，以及不同邻居数/半径对聚类质量的影响。
2. 模型结构优化：调整 recurrent GNN 层数、hidden size、message passing 次数、残差连接和归一化方式。
3. 训练目标优化：比较坐标变换损失、边关系监督、对比学习目标，以及与最终聚类指标更一致的 surrogate loss。
4. 数据增强优化：系统测试定位噪声、点密度、cluster shape、多尺度混合场景下的鲁棒性。
5. 聚类后处理优化：在 MIRO 输出后自动搜索 DBSCAN 的 eps/min_samples，或比较 HDBSCAN 等替代聚类器。
6. 成本约束优化：记录训练/推理时间和显存占用，优先寻找在同等或更低计算成本下提升指标的方案。

## 阶段性验收标准

1. 最小复现：baseline DBSCAN 与 MIRO pipeline 都能在同一数据划分上端到端运行。
2. 指标对齐：复现实验的核心指标趋势应与论文一致，即 MIRO + DBSCAN 明显优于直接 DBSCAN。
3. 自动优化：agent 至少完成一轮提出方案、修改实现/配置、运行实验、比较指标、写出结论的闭环。
4. 可追溯：每个实验结果都能追溯到代码版本、配置文件、数据划分和随机种子。
