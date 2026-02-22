# NCFI Freight Rate Index Prediction

> 宁波美西航线集装箱运价指数（NCFI）预测 —— 基于多源供应链数据的机器学习与时间序列建模

---

## 项目简介

本项目旨在对**宁波集装箱运价指数（Ningbo Containerized Freight Index, NCFI）美西航线**进行预测分析。通过整合洛杉矶港口运营数据、宏观经济指标、燃油价格及中国制造业景气指数等多维供应链数据，构建并对比多种预测模型，探索影响跨太平洋集运市场运价波动的关键因素。

---

## 数据来源

| 文件 | 内容 | 时间范围 |
|------|------|----------|
| `Table1_只保留中国制造业PMI_月度数据(2014-2024).xlsx` | 中国制造业 PMI 月度数据 | 2014–2024 |
| `Table2_LA LB_CPI.xlsx` | 洛杉矶-长滩地区消费者价格指数 (CPI) | — |
| `Table3_Diesel_Price(2004-2024).xlsx` | 美国柴油价格（美分/加仑） | 2004–2024 |
| `Table4_LA_Container_TEUs.xlsx` | 洛杉矶港口集装箱吞吐量（TEUs） | 2012–2024 |
| `Table5_美西运价指数NCFI(2014-2024).xlsx` | 宁波集装箱运价指数（目标变量） | 2014–2024 |
| `Table6_LA_Vessel_Activity.xlsx` | 洛杉矶港船舶活动数据（锚泊、靠泊、离港等） | 2015–2024 |

---

## 特征工程

- **滞后特征**：NCFI 过去 1、2、3 个月的值（`NCFI_lag1/2/3`）
- **移动平均**：3 个月与 12 个月滚动均值（`NCFI_MA3`, `NCFI_MA12`）
- **STL 季节性分解**：趋势项、季节项、残差项（`trend_stl`, `seasonal_stl`, `residual_stl`）
- **节假日标记**：美国联邦节假日月度标记（`Federal_Holiday`）
- **时间特征**：月份、季度

---

## 模型方法

### 1. 线性回归（基准模型）
- `sklearn.LinearRegression`
- PyTorch 实现的线性回归（用于对照验证）

### 2. 随机森林
- `sklearn.RandomForestRegressor`
- 采用时间序列交叉验证（`TimeSeriesSplit`, 10 折）
- 基于特征重要性得分进行特征分析

### 3. SARIMAX
- 使用 `pmdarima.auto_arima` 自动调优 $(p,d,q)(P,D,Q)_m$ 参数
- 引入外生变量：柴油价格、锚泊船舶数
- 动态滚动单步预测（Rolling Forecast）

### 4. LSTM（长短期记忆网络）
- PyTorch 实现，序列窗口长度为 3
- 特征标准化（`StandardScaler`）
- 学习率自适应调度（`ReduceLROnPlateau`）
- 梯度法特征重要性可视化

---

## 评估指标

| 指标 | 说明 |
|------|------|
| RMSE | 均方根误差 |
| R² | 决定系数 |
| MAE | 平均绝对误差 |
| MAPE | 平均绝对百分比误差 |
| DA | 方向准确性（涨跌方向预测正确率） |

---

## 项目结构

```
NCFI-prediction-main/
├── NCFI_Prediction.ipynb      # 主分析 Notebook
├── README.md
├── Table1_只保留中国制造业PMI_月度数据(2014-2024).xlsx
├── Table2_LA LB_CPI.xlsx
├── Table3_Diesel_Price(2004-2024).xlsx
├── Table4_LA_Container_TEUs.xlsx
├── Table5_美西运价指数NCFI(2014-2024).xlsx
├── Table6_LA_Vessel_Activity.xlsx
└── merged_data_cleaned_final.csv   # 清洗合并后的数据（运行后生成）
```

---

## 环境依赖

```bash
pip install pandas numpy scikit-learn statsmodels pmdarima torch matplotlib seaborn openpyxl
```

---

## 快速开始

1. 将所有 Excel 数据文件放置于 `NCFI_Prediction.ipynb` 同目录下
2. 按顺序运行 Notebook 各单元格
3. 数据预处理完成后会生成 `merged_data_cleaned_final.csv`

---

## 主要结论

- **SARIMAX** 在引入外生变量与自动参数调优后，预测效果最为稳定，适合短期动态滚动预测。
- **随机森林** 在 2021–2022 年运价剧烈波动期预测误差较大，反映树模型对极端趋势外推能力有限。
- **LSTM** 经过参数调优后表现良好，梯度重要性分析与随机森林特征重要性结果基本一致。
- 滞后特征（`NCFI_lag1`）与移动平均特征对预测贡献最大；宏观指标（PMI、柴油价格）提供辅助解释力。

---

## 许可证

本项目仅供学术研究与课程作业使用。
