# 城市安全感知建模

Gu Rui · UCL Bartlett · MSc Architectural Computation 毕业论文 · 2025

[English README](README.md) · [数据清单](docs/DATA.md) · [审核报告](docs/AUDIT.md)

本项目结合 Place Pulse 2.0 的安全感知评分、Space Syntax 街道网络指标，以及 SegFormer 街景语义分割，研究伦敦街道环境与安全感知之间的关联。本地建模表有 900 个唯一地点。

## 从哪里开始

- 主要分析入口：`02_DataAnalysis/MergedData_Model_V2.ipynb`。
- 仅空间指标的基线：`02_DataAnalysis/SS_Model_V1.ipynb`。
- 数据准备、空间匹配和图像分割：`01_DataProcessing/` 下的三份 notebook。
- 模型结果：`results/model_comparison_results.csv`。
- 数据来源、字段和运行前准备：[DATA.md](docs/DATA.md)。

## 已核实的结果

| 模型 | 特征数，不含截距 | 样本内 R² |
| --- | ---: | ---: |
| 四变量线性模型 | 4 | 0.017405 |
| 二次多项式模型 | 14 | 0.080896 |
| 四变量 + 六个两两交互项 | 10 | 0.034764 |

2026-10-08 使用本地数据独立复算了以上 R²，与历史结果一致。它们是样本内拟合结果，不能据此宣称具有样本外预测能力或因果关系。

## 整理说明

此次整理保留原始研究代码，清除 notebook 已保存输出，重新编写 README，并将已有 CSV 区分为历史数据和结果。 随后的 GitHub 清理将 `data/legacy/` 和 `results/legacy/` 中四份旧 CSV 取消追踪，文件仍完整保留在本地，并加入忽略规则；历史提交不改写。完整图片、GIS 数据、论文草稿和讨论文件仍保留在本地论文目录。

原始 notebook 仍有硬编码路径；数据合并步骤未纳入代码；最终空间匹配使用经纬度坐标，需要在重新计算前检查。当前仓库属于有文档的研究档案，还不能直接下载后一键复现。详细问题与验证范围见 [审核报告](docs/AUDIT.md)。

运行依赖和操作顺序见 [English README](README.md)。
