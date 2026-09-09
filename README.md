# SVMD-DRMT: Bearing Remaining Useful Life Prediction

## SVMD-DRMT：滚动轴承剩余寿命预测

**A Successive Variational Mode Decomposition-Assisted Domain-Regularized Multi-Scale Transformer for Bearing RUL Prediction**  
**基于连续变分模态分解与域正则化多尺度 Transformer 的轴承剩余寿命预测框架**

<p align="center">
  <img src="https://img.shields.io/badge/Task-Bearing%20RUL%20Prediction-1f6feb?style=flat-square" alt="task badge" />
  <img src="https://img.shields.io/badge/Model-Multi--Scale%20Transformer-7c3aed?style=flat-square" alt="model badge" />
  <img src="https://img.shields.io/badge/Signal-SVMD%20Feature%20Processing-f97316?style=flat-square" alt="signal badge" />
  <img src="https://img.shields.io/badge/Interpretability-SHAP-059669?style=flat-square" alt="interpretability badge" />
</p>

> This repository is a paper-oriented showcase of SVMD-DRMT. It documents the degradation-feature pipeline, model architecture, source-bearing regularization, evaluation protocol and interpretability evidence.  
> 本仓库是 SVMD-DRMT 的论文展示型主页，用于介绍退化特征处理流程、模型架构、源轴承正则化、实验协议和可解释性结果。

> **Code status / 代码状态:** The repository currently contains the paper figures and research documentation. The complete implementation will be released or expanded in a later update.  
> 当前仓库主要包含论文图示和研究说明，完整实现将在后续整理后进一步发布或补充。

---

## Research motivation / 研究动机

Remaining useful life (RUL) prediction for rolling bearings is difficult because vibration-derived degradation features are non-stationary, noisy and heterogeneous across individual bearings—even under the same rotational speed and load. A model that fits one bearing well may still fail to generalize to another bearing with a different degradation trajectory.  
滚动轴承剩余寿命（RUL）预测之所以困难，是因为振动信号提取的退化特征具有非平稳性、噪声干扰和个体差异。即使转速和载荷完全相同，不同轴承的退化轨迹也可能明显不同，一个轴承上拟合良好的模型不一定能够可靠地泛化到另一个轴承。

SVMD-DRMT addresses this problem through a connected research chain:

SVMD-DRMT 通过以下完整技术链路应对这一问题：

**trend stabilization → multi-scale local representation → global temporal modeling → source-bearing representation regularization → degradation-order constraint → interpretable feature attribution**  
**趋势稳定化 → 多尺度局部表示 → 全局时序建模 → 源轴承表示正则化 → 退化顺序约束 → 可解释特征归因**

The goal is not to claim universal online prognostics. The reported study focuses on **offline, same-condition, cross-bearing RUL prediction** under PRONOSTIA Operating Condition 1.  
本文并不声称已经解决通用在线预测问题，而是聚焦于 PRONOSTIA 工况 1 下的**离线、同工况、跨轴承 RUL 预测**。

![SVMD-assisted degradation feature representation process](https://cdn.jsdelivr.net/gh/Minamoto-Chisei-Emperor/SVMD-DRMT@349cf4e/assets/figures/01_svmd_feature_process.png)

*From raw vibration to stabilized degradation trajectories / 从原始振动到稳定退化轨迹*

---

## Core idea / 核心思想

The method separates the problem into two complementary parts:

该方法将问题拆分为两个相互衔接的部分：

1. **Make the input trajectory more reliable / 让输入退化轨迹更可靠**  
   Extract 33 multi-domain features and process each full-life feature trajectory with SVMD-assisted mode selection and reconstruction.  
   提取 33 维多域特征，并对每个轴承的全寿命特征轨迹进行基于 SVMD 的模态筛选与重构。
2. **Make the temporal predictor more discriminative and transferable / 让时序预测器更具区分性和泛化性**  
   Use multi-scale temporal convolutions and a Transformer to model local-to-global degradation dynamics, then regularize source-bearing representations and impose a soft RUL order constraint.  
   使用多尺度时序卷积和 Transformer 建模从局部到全局的退化动态，再通过源轴承表示正则化和软 RUL 顺序约束改善跨轴承预测。

![Overall SVMD-DRMT workflow](https://cdn.jsdelivr.net/gh/Minamoto-Chisei-Emperor/SVMD-DRMT@349cf4e/assets/figures/02_overall_workflow.png)

*End-to-end research workflow / 端到端研究流程*

---

## Data and evaluation protocol / 数据与评价协议

The experiments use the publicly available IEEE PHM 2012 FEMTO Bearing Dataset collected on the PRONOSTIA accelerated degradation platform. The study selects **Bearing1_1–Bearing1_7 under Operating Condition 1**, with a rotational speed of 1,800 r/min and a radial load of 4,000 N. Each vibration recording contains 2,560 samples at 25.6 kHz, and the present study uses the horizontal vibration channel.  
实验采用公开的 IEEE PHM 2012 FEMTO Bearing Dataset，该数据由 PRONOSTIA 轴承加速退化实验平台采集。研究选取**工况 1 下的 Bearing1_1–Bearing1_7**，转速为 1,800 r/min，径向载荷为 4,000 N。每次振动记录包含 2,560 个采样点，采样频率为 25.6 kHz，本文仅使用水平振动通道。

### Leave-one-bearing-out design / 留一轴承交叉验证

Each of the seven bearings is held out once as the test bearing, while the remaining six bearings provide the source-bearing training domains. Results are averaged over seven rounds. This protocol measures same-condition cross-bearing generalization, but the current implementation remains an offline, target-aware evaluation because complete unlabeled test-bearing feature statistics and test labels are used for scheduling, early stopping and model selection.  
7 个轴承分别轮流作为测试轴承，其余 6 个轴承作为源轴承训练域，共进行 7 轮实验并取平均结果。该协议用于评价同工况跨轴承泛化能力，但当前实验仍属于离线、目标感知型评价：测试轴承的完整无标签特征统计量以及测试标签被用于学习率调度、早停和模型选择。

![Leave-one-bearing-out evaluation protocol](https://cdn.jsdelivr.net/gh/Minamoto-Chisei-Emperor/SVMD-DRMT@349cf4e/assets/figures/05_loo_protocol.png)

*Seven-fold leave-one-bearing-out protocol / 七折留一轴承评价协议*

### Multi-domain input / 多域输入特征

| Domain | Count | English | 中文 |
|---|---:|---|---|
| Time domain | 18 | Amplitude, impact, dispersion and entropy statistics. | 描述振幅、冲击、离散程度和熵特征。 |
| Hilbert domain | 4 | Instantaneous-amplitude and normalized instantaneous-frequency statistics. | 描述瞬时幅值与归一化瞬时频率。 |
| Frequency domain | 11 | Spectral shape, entropy and four frequency-band energy ratios. | 描述频谱形状、频谱熵以及四个频带能量比例。 |
| **Total** | **33** | Complementary degradation descriptors. | 互补的退化特征描述。 |

![SVMD feature-process figure](https://cdn.jsdelivr.net/gh/Minamoto-Chisei-Emperor/SVMD-DRMT@349cf4e/assets/figures/01_svmd_feature_process.png)

---

## Method architecture / 方法架构

![Detailed SVMD-DRMT architecture](https://cdn.jsdelivr.net/gh/Minamoto-Chisei-Emperor/SVMD-DRMT@349cf4e/assets/figures/03_model_architecture.png)

*Detailed model architecture / 模型详细架构*

### 1. SVMD-assisted trend processing / SVMD 辅助趋势处理

SVMD is applied to the **full-life trajectory of each engineered feature**, not directly to every short raw-vibration window. Modes are extracted sequentially from the residual, and reconstruction components are selected using center frequency, relative modal energy and absolute correlation with the original trajectory. This stabilizes feature trajectories while avoiding indiscriminate removal of degradation-related local components.  
SVMD 作用于**每个工程特征的全寿命轨迹**，而不是直接对每个短时原始振动窗口进行分解。算法从残差中逐步提取模态，并依据中心频率、模态相对能量以及与原始轨迹的绝对相关性筛选重构分量，从而稳定退化趋势，同时避免简单地删除携带退化信息的局部成分。

The reported SVMD settings include a bandwidth penalty of 2000, at most 6 modes, 200 iterations per mode, convergence tolerance 10⁻⁶, residual-energy threshold 0.03, maximum retained normalized center frequency 0.25, minimum modal energy ratio 0.001 and minimum absolute correlation 0.08.  
论文报告的 SVMD 参数包括：带宽惩罚系数 2000、最多 6 个模态、每个模态最多迭代 200 次、收敛容差 10⁻⁶、残差能量阈值 0.03、最大保留归一化中心频率 0.25、最小模态能量比例 0.001 以及最小绝对相关系数 0.08。

### 2. Multi-scale temporal representation / 多尺度时序表示

Three parallel depthwise temporal-convolution branches with kernel sizes **3, 5 and 7** capture short-range changes at different local scales. Pointwise fusion, Group Normalization, SiLU, dropout and a residual path combine the branches into a 48-dimensional hidden representation.  
三个并行的深度可分离时序卷积分支使用 **3、5、7** 的卷积核，捕获不同局部尺度下的短期变化。随后通过逐点卷积融合、Group Normalization、SiLU、dropout 和残差路径，形成 48 维隐藏表示。

### 3. Transformer global temporal encoding / Transformer 全局时序编码

The fused local representation is passed to a two-layer pre-norm Transformer encoder with four attention heads, learnable positional embeddings and a feed-forward dimension of 128. Self-attention connects non-adjacent time positions within the sliding window, complementing the local inductive bias of the convolutions.  
融合后的局部表示输入两层 Pre-Norm Transformer 编码器，使用 4 个注意力头、可学习位置嵌入以及 128 维前馈网络。自注意力建立滑动窗口内非相邻时间位置之间的联系，与卷积提供的局部归纳偏置形成互补。

### 4. Channel recalibration and adaptive dual-path pooling / 通道重校准与自适应双路径池化

Channel gating recalibrates the hidden dimensions. The temporal output is then summarized through two paths: global average pooling describes the overall degradation state, while temporal attention pooling emphasizes informative time points. A vector gate adaptively fuses the two representations before the 48→96→48→1 regression head.  
通道门控机制重新调整隐藏通道的重要性。随后采用双路径聚合：全局平均池化描述整体退化状态，时间注意力池化强调信息量更高的时刻，最后通过向量门控自适应融合两类表示，并送入 48→96→48→1 回归头。

![Joint optimization mechanism](https://cdn.jsdelivr.net/gh/Minamoto-Chisei-Emperor/SVMD-DRMT@349cf4e/assets/figures/04_joint_optimization.png)

*Architecture, joint losses and SHAP interpretation / 架构、联合损失与 SHAP 解释*

---

## Joint optimization objective / 联合优化目标

SVMD-DRMT does not rely on a point-wise regression loss alone. Its total objective combines three terms:

SVMD-DRMT 不仅依赖逐样本回归误差，而是联合优化三个目标：

| Term | English role | 中文作用 |
|---|---|---|
| Smooth L1 regression | Fits normalized RUL targets and limits sensitivity to outliers. | 拟合归一化 RUL，并降低异常值的影响。 |
| Source-bearing covariance regularization | Reduces covariance differences among deep representations from training bearings. | 减小不同训练轴承深层表示之间的协方差差异。 |
| Order-consistency loss | Softly penalizes local violations of the expected non-increasing RUL order. | 对违反 RUL 非递增退化顺序的局部预测进行软惩罚。 |

The covariance term is **source-bearing regularization**, not unsupervised target-domain adaptation: the test bearing is not directly aligned in the loss. The order loss does not force every adjacent prediction to decrease; it only discourages clearly non-physical local increases.  
其中协方差项属于**源轴承正则化**，并不是无监督目标域适配：测试轴承不会直接参与该损失中的分布对齐。顺序损失也不会强制每一个相邻预测点都下降，而是仅抑制明显不符合退化规律的局部上升。

---

## Comparative results / 对比实验结果

The paper compares SVMD-DRMT with Transformer, Informer, GRU, ConvLSTM and PatchTST across seven leave-one-bearing-out experiments.  
论文在 7 轮留一轴承实验中，将 SVMD-DRMT 与 Transformer、Informer、GRU、ConvLSTM 和 PatchTST 进行比较。

| Model | MSE ↓ | RMSE ↓ | MAE ↓ | R² ↑ | MAPE (%) ↓ | Score ↓ |
|---|---:|---:|---:|---:|---:|---:|
| PatchTST | 0.0099 | 0.0963 | 0.0756 | 0.8800 | 27.1302 | 13.4327 |
| Informer | 0.0100 | **0.0953** | 0.0740 | 0.8784 | 28.8910 | 13.1860 |
| Transformer | 0.0112 | 0.1003 | 0.0764 | 0.8644 | 30.7768 | 13.6645 |
| ConvLSTM | 0.0131 | 0.1121 | 0.0904 | 0.8416 | 32.3515 | 15.8732 |
| GRU | 0.0241 | 0.1542 | 0.1300 | 0.7083 | 37.6233 | 22.2053 |
| **SVMD-DRMT** | 0.0106 | 0.0995 | **0.0728** | 0.8716 | 30.1314 | **12.8685** |

SVMD-DRMT does not win every conventional metric. Its clearest advantages are the **lowest mean MAE** and **lowest asymmetric Score**, while Informer achieves the lowest mean RMSE and PatchTST achieves the lowest mean MSE, highest mean R² and lowest MAPE. This pattern is important: the contribution of SVMD-DRMT is best understood as a balanced, risk-sensitive and interpretable predictor rather than a universal winner on every metric.  
SVMD-DRMT 并非在所有传统指标上都排名第一。它最突出的优势是取得**最低平均 MAE**和**最低非对称 Score**；Informer 的平均 RMSE 最低，PatchTST 的平均 MSE、平均 R² 和 MAPE 更优。由此可见，SVMD-DRMT 更适合被理解为一个兼顾绝对误差、风险敏感性和可解释性的平衡型预测器，而不是在每一项指标上都绝对领先的模型。

![Overall model comparison](https://cdn.jsdelivr.net/gh/Minamoto-Chisei-Emperor/SVMD-DRMT@349cf4e/assets/figures/06_model_comparison.jpg)

*Mean metrics and variability across seven LOO experiments / 七轮留一轴承实验的平均指标与波动*

---

## Bearing-level behavior / 轴承级行为

Different bearings exhibit different prediction difficulty. Bearing1_3 is relatively regular, Bearing1_4 shows intermediate deviations, and Bearing1_7 is more challenging with stronger local fluctuations. SVMD-DRMT performs close to the best models on regular cases and shows a particularly favorable result on Bearing1_7 under the reported comparison.  
不同轴承具有不同的预测难度。Bearing1_3 的退化轨迹相对规则，Bearing1_4 存在中等程度偏差，而 Bearing1_7 的局部波动更明显、预测难度更高。在规则样本上，SVMD-DRMT 与最优模型接近；在论文对比中，它在 Bearing1_7 上表现尤其突出。

![Representative-bearing predictions and error distributions](https://cdn.jsdelivr.net/gh/Minamoto-Chisei-Emperor/SVMD-DRMT@349cf4e/assets/figures/07_representative_bearings.jpg)

*Representative bearings: Bearing1_3, Bearing1_4 and Bearing1_7 / 代表性轴承：Bearing1_3、Bearing1_4 与 Bearing1_7*

![Asymmetric Score heatmap](https://cdn.jsdelivr.net/gh/Minamoto-Chisei-Emperor/SVMD-DRMT@349cf4e/assets/figures/08_score_heatmap.jpg)

*Raw cumulative Score for within-bearing model comparison / 用于轴承内模型比较的原始累积 Score*

The heatmap should be interpreted within each bearing because different bearings contain different numbers of test windows. The seven-fold average Score in the comparison table is the appropriate summary of overall risk-sensitive performance.  
由于不同轴承包含的测试窗口数量不同，热力图应主要用于同一轴承内部的模型比较；整体风险敏感性能应以对比表中的七折平均 Score 为准。

![Multi-bearing RUL prediction trajectories](https://cdn.jsdelivr.net/gh/Minamoto-Chisei-Emperor/SVMD-DRMT@349cf4e/assets/figures/09_bearing_trajectories.jpg)

*Prediction trajectories across Bearing1_1–Bearing1_7 / Bearing1_1–Bearing1_7 的预测轨迹*

![Bearing-level metric comparisons](https://cdn.jsdelivr.net/gh/Minamoto-Chisei-Emperor/SVMD-DRMT@349cf4e/assets/figures/10_bearing_metrics.jpg)

*Six metrics at the bearing level / 轴承级六项指标对比*

---

## SHAP-based interpretation / 基于 SHAP 的模型解释

SHAP analysis is performed on 496 test samples containing all 33 multi-domain features. The most influential feature is `band_energy_ratio_2` (mean absolute SHAP value 0.017254), followed by `hilbert_frequency_mean` (0.006432) and `spectral_spread` (0.005438). Frequency-band energy redistribution, instantaneous-frequency structure and spectral-shape descriptors dominate the learned feature-use pattern, while time-domain statistics provide complementary information.  
SHAP 分析基于包含全部 33 个多域特征的 496 个测试样本。影响最大的特征是 `band_energy_ratio_2`（平均绝对 SHAP 值 0.017254），其次为 `hilbert_frequency_mean`（0.006432）和 `spectral_spread`（0.005438）。模型主要利用频带能量重分布、瞬时频率结构和频谱形状信息，同时结合时域统计特征作为补充。

![SHAP interpretation](https://cdn.jsdelivr.net/gh/Minamoto-Chisei-Emperor/SVMD-DRMT@349cf4e/assets/figures/11_shap_interpretation.jpg)

*Global importance, beeswarm distribution, dependence and local explanation / 全局重要性、蜂群图、依赖关系与局部解释*

SHAP values describe statistical attribution inside the trained model; they should not be interpreted as direct physical causality between an individual feature and bearing damage.  
SHAP 值描述的是训练模型内部的统计归因关系，不能直接等同于某个特征导致轴承损伤的物理因果关系。

---

## Ablation study / 消融实验

| Configuration | MSE | RMSE | MAE | R² | MAPE (%) | Score | 中文说明 |
|---|---:|---:|---:|---:|---:|---:|---|
| Full SVMD-DRMT | 0.0106 | 0.0995 | **0.0728** | **0.8716** | 30.1314 | **12.8685** | 完整模型 |
| w/o SVMD | 0.0106 | 0.0995 | 0.0728 | 0.8705 | 30.1314 | 12.9768 | 不进行 SVMD 趋势处理 |
| w/o Multi-scale | **0.0102** | **0.0975** | 0.0764 | 0.8667 | **28.5311** | 13.5292 | 单一卷积尺度 |
| w/o Domain Reg. | 0.0106 | 0.0993 | 0.0736 | 0.8621 | 30.0623 | 12.8936 | 去除源轴承协方差正则 |
| w/o Order Loss | 0.0106 | 0.0995 | 0.0728 | 0.8617 | 30.1232 | 12.9658 | 去除顺序一致性损失 |
| Mean Pooling Only | 0.0122 | 0.1066 | 0.0765 | 0.8524 | 32.1019 | 13.7573 | 仅使用平均池化 |

The clearest ablation result comes from replacing adaptive dual-path pooling with mean pooling only: R² decreases from 0.8716 to 0.8524 and Score increases from 12.8685 to 13.7573. Removing the source-bearing regularizer or order loss mainly affects representation consistency and risk-sensitive behavior rather than uniformly reducing every point-wise error. The multi-scale module shows a genuine trade-off: the single-scale variant improves squared-error metrics but worsens MAE, R² and Score.  
最明显的消融结果来自将自适应双路径池化替换为单纯平均池化：R² 从 0.8716 降至 0.8524，Score 从 12.8685 升至 13.7573。去除源轴承正则项或顺序损失，主要影响表示一致性和风险敏感行为，而不是让所有逐点误差指标同步恶化。多尺度模块体现出真实的权衡：单尺度变体的平方误差指标更好，但 MAE、R² 和 Score 变差。

---

## Model complexity and scope / 模型复杂度与适用范围

SVMD-DRMT contains 74,643 trainable parameters, substantially more than the baseline Transformer (1,625), Informer (5,569), GRU (7,137), ConvLSTM (10,985) and PatchTST (20,897). The method therefore prioritizes richer degradation representation, cross-bearing regularization and risk-sensitive accuracy over lightweight deployment.  
SVMD-DRMT 包含 74,643 个可训练参数，明显高于基线 Transformer（1,625）、Informer（5,569）、GRU（7,137）、ConvLSTM（10,985）和 PatchTST（20,897）。因此，该方法优先追求更丰富的退化表示、跨轴承正则化和风险敏感预测精度，而不是轻量化部署。

The current study has three important boundaries: single operating condition, target-aware offline evaluation, and SVMD processing over full-life feature trajectories. These choices are appropriate for the paper’s methodological study but should not be presented as causal online prognostics or strictly target-unseen domain generalization.  
当前研究有三个重要边界：单一工况、目标感知型离线评价，以及基于全寿命特征轨迹的 SVMD 处理。这些设置适合论文中的方法学研究，但不应被表述为因果在线预测或严格的目标不可见域泛化。

---

## Repository contents / 仓库内容

```text
SVMD-DRMT/
├── README.md
├── Figure/
│   └── 高清图片/          # Original high-resolution paper figures
└── assets/
    └── figures/            # Stable display assets used by this README
```

- **Research documentation / 研究说明:** complete bilingual explanation of the paper’s method and evidence.  
  提供论文方法与实验依据的完整双语说明。
- **Figure assets / 图示资源:** workflow, architecture, evaluation, comparison, SHAP and ablation visuals.  
  包含流程图、架构图、评价协议、对比实验、SHAP 和消融图示。
- **Code / 代码:** not yet presented as an end-to-end executable package.  
  当前尚未作为端到端可运行软件包发布。

---

## Planned reproduction workflow / 计划中的复现流程

When the implementation is released, the intended sequence is:

代码正式发布后，计划按照以下顺序复现：

1. Load PRONOSTIA Bearing1_1–Bearing1_7 under Operating Condition 1 and use the horizontal vibration channel.  
   加载工况 1 下 Bearing1_1–Bearing1_7，并使用水平振动通道。
2. Extract the 18 time-domain, 4 Hilbert-domain and 11 frequency-domain features.  
   提取 18 个时域、4 个 Hilbert 域和 11 个频域特征。
3. Apply SVMD-assisted full-life trend processing and reconstruct the selected components.  
   对全寿命特征轨迹进行 SVMD 辅助趋势处理并重构筛选后的模态。
4. Standardize features per bearing and construct sliding windows.  
   按轴承标准化特征并构造滑动窗口。
5. Train SVMD-DRMT with regression, source-bearing covariance and order-consistency losses.  
   使用回归损失、源轴承协方差正则和顺序一致性损失训练 SVMD-DRMT。
6. Evaluate MSE, RMSE, MAE, R², MAPE and asymmetric Score, then generate SHAP explanations.  
   计算 MSE、RMSE、MAE、R²、MAPE 和非对称 Score，并生成 SHAP 解释。

---

## Paper information / 论文信息

- **Title / 题目:** *SVMD-DRMT: A Domain-Regularized Multi-Scale Transformer for Bearing Remaining Useful Life*  
- **Authors / 作者:** Shiyang Li, Shuqi Zhu, Yan Gao  
- **Dataset / 数据集:** IEEE PHM 2012 FEMTO Bearing Dataset (PRONOSTIA)  
- **Repository / 项目:** [Minamoto-Chisei-Emperor/SVMD-DRMT](https://github.com/Minamoto-Chisei-Emperor/SVMD-DRMT)

### Citation / 引用

```bibtex
@misc{li_svmd_drmt,
  title  = {SVMD-DRMT: A Domain-Regularized Multi-Scale Transformer for Bearing Remaining Useful Life},
  author = {Li, Shiyang and Zhu, Shuqi and Gao, Yan},
  note   = {Research repository for bearing RUL prediction}
}
```

---

## License and responsible use / 许可与负责任使用

Please follow the original FEMTO/PRONOSTIA dataset terms and the repository license when using or redistributing the materials. The results are intended for academic research and methodological demonstration, not direct safety-critical maintenance decisions without independent validation.  
使用或重新分发相关材料时，请遵循 FEMTO/PRONOSTIA 数据集原始条款和仓库许可证。本文结果主要用于学术研究与方法展示，未经独立验证，不应直接用于安全关键型维护决策。
