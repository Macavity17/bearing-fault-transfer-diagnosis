# 高速列车轴承智能故障诊断（华为杯 2025）

基于 CWRU 轴承振动数据的**源域故障诊断**与**跨域迁移诊断**。源域为带标签的实验台信号，
目标域为 16 组无标签信号（A–P），通过 CORAL 域对齐把源域模型迁移到目标域，并用 SHAP 解释诊断依据。

## 技术路线

```
原始振动信号 (.mat)
      │
      ├─ 时域/频域基础特征 (Mean/RMS/Kurt/CF/FreqSkew …)
      └─ 高级特征工程 (边带能量与比值、包络谱 BPFO/BPFI/BSF 峰值)
      │
      ├── 源域 ──→ SMOTE 增强 ──→ 标准化/划分 ──→ RandomForest / XGBoost ──→ 诊断
      │                                                     │
      └── 目标域 ──→ CORAL 协方差对齐 ──→ SMOTENC 增强 ──→ 迁移诊断 ──→ SHAP 可解释性
```

## 目录结构

| 目录 | 内容 |
|---|---|
| `docs/` | 赛题、论文（doc/pdf）、`变量说明.txt`（全部特征字段的中文释义） |
| `data/raw/source/` | 源域原始信号 161 个 `.mat`，按 `采样率_测点 / 故障类型 / 损伤尺寸` 分层 |
| `data/raw/target/` | 目标域原始信号 `A.mat`–`P.mat`，以及展平后的 `目标域.csv` |
| `data/features/` | 特征表。`source/` 源域、`target/` 目标域，根目录 `dataset_complete.csv` 为两域合并表（CORAL 输入） |
| `data/processed/` | 标准化 + 训练/测试划分后的 pickle 数据包，供训练 notebook 直接加载 |
| `notebooks/` | 按流水线编号的 15 个 notebook，见下表 |
| `models/` | 训练好的模型。`source/` 源域 RF+XGBoost、`target/` 迁移后 RF |
| `results/` | 全部图表与运行日志，按问题分区 |
| `_归档/` | **可删除**：重复文件、旧版 notebook、改路径前的备份。详见文末 |

### notebooks 执行顺序

| 编号 | 文件 | 作用 |
|---|---|---|
| 00 | `00_全流程汇总.ipynb` | 端到端复现全流程（其余 notebook 的汇总版） |
| 01 | `01_信号滤波与包络谱分析.ipynb` | 带通滤波、包络解调、故障特征频率验证 |
| 10 → 14 | `源域_基础特征提取` → `高级特征工程` → `数据增强` → `预处理` → `模型训练` | **问题一：源域故障诊断** |
| 20 → 26 | `目标域_基础特征提取` → `高级特征工程` → `CORAL域对齐` → `数据增强` → `预处理` → `迁移诊断_模型训练` → `SHAP可解释性分析` | **问题二：跨域迁移诊断** |
| 90 | `90_数据速查.ipynb` | 查看 `.mat` 内部变量名的小工具 |

### results 分区

- `01_源域诊断/` — 混淆矩阵、特征重要性、交叉验证、模型对比图 + 三份运行日志
- `02_迁移诊断/` — CORAL 前后 t-SNE / 分布对比、特征选择各阶段模型对比，含两个子专题目录
- `shap/` — SHAP 汇总图与 15 张分类别 beeswarm 图
- `滤波分析/` — 频谱/包络谱图与各测点 FFT 数据
- `预测结果/` — `target_domain_predictions.csv`，目标域 A–P 的最终诊断结果
- `00_全流程/` — 空目录，`00_全流程汇总.ipynb` 重跑时的产物落点（不覆盖上面已归档的结果）

## 运行方式

所有 notebook 均以 **`notebooks/` 为工作目录**，内部使用 `../data/...`、`../results/...` 相对路径，
在 `notebooks/` 下启动 Jupyter 即可直接运行，无需修改任何路径。

依赖见 `requirements.txt`（原始环境为 Anaconda base / Python 3.11.7）：

```bash
pip install -r requirements.txt
cd notebooks && jupyter lab
```
