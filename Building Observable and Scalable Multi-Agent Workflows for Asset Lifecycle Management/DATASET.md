Download: https://www.kaggle.com/datasets/behrad3d/nasa-cmaps

# NASA C-MAPSSData Dataset 介紹

## 1. Dataset 概述

NASA C-MAPSSData 是一個用於 **Remaining Useful Life, RUL 預測** 的飛機渦輪風扇引擎資料集。

資料由 NASA C-MAPSS 引擎模擬器產生，模擬多台 aircraft turbofan engine 從正常運轉、逐漸劣化，到最終失效的過程。每一台 engine 會形成一條 multivariate time series，每個 row 代表該 engine 在某個 operational cycle 的感測器與操作狀態快照。

此 dataset 常用於：

- Predictive Maintenance
- Prognostics and Health Management, PHM
- Remaining Useful Life, RUL Prediction
- Time-series Regression
- Failure Prediction

---

## 2. 任務目標

此 dataset 的核心任務是：

> 根據 engine 的歷史 sensor measurements，預測該 engine 距離 failure 還剩下多少 operational cycles。

也就是預測：

```text
Remaining Useful Life, RUL
````

在 training set 中，每台 engine 的資料會一路記錄到 failure。

在 test set 中，每台 engine 的資料會在 failure 前被截斷，模型需要根據最後觀測到的 sensor sequence 預測剩餘壽命。

---

## 3. Dataset 組成

C-MAPSSData 包含四個子資料集：

| Dataset | Operating Conditions | Fault Modes | 說明                |
| ------- | -------------------: | ----------: | ----------------- |
| FD001   |                  1 種 |         1 種 | 最簡單，單一操作條件與單一故障模式 |
| FD002   |                  6 種 |         1 種 | 多操作條件，單一故障模式      |
| FD003   |                  1 種 |         2 種 | 單一操作條件，多故障模式      |
| FD004   |                  6 種 |         2 種 | 最困難，多操作條件與多故障模式   |

其中：

```text
FD001: Sea Level + HPC Degradation
FD002: Six Conditions + HPC Degradation
FD003: Sea Level + HPC Degradation + Fan Degradation
FD004: Six Conditions + HPC Degradation + Fan Degradation
```

---

## 4. 檔案結構

每個子資料集包含三種檔案：

```text
train_FDxxx.txt
test_FDxxx.txt
RUL_FDxxx.txt
```

例如 FD001 包含：

```text
train_FD001.txt
test_FD001.txt
RUL_FD001.txt
```

各檔案用途如下：

| File              | 說明                                       |
| ----------------- | ---------------------------------------- |
| `train_FDxxx.txt` | Training data，每台 engine 從正常狀態記錄到 failure |
| `test_FDxxx.txt`  | Test data，每台 engine 在 failure 前被截斷       |
| `RUL_FDxxx.txt`   | Test data 中每台 engine 最後一個 cycle 後的真實 RUL |

---

## 5. Dataset 規模

根據目前資料檔案統計：

| Dataset | Train Units | Test Units | Train Rows | Test Rows | RUL Rows |
| ------- | ----------: | ---------: | ---------: | --------: | -------: |
| FD001   |         100 |        100 |     20,631 |    13,096 |      100 |
| FD002   |         260 |        259 |     53,759 |    33,991 |      259 |
| FD003   |         100 |        100 |     24,720 |    16,596 |      100 |
| FD004   |         249 |        248 |     61,249 |    41,214 |      248 |

---

## 6. Data Format

每個 `train_FDxxx.txt` 與 `test_FDxxx.txt` 檔案共有 26 個 columns。

每個 row 是某一台 engine 在某一個 cycle 的狀態快照。

| Column | 名稱                       | 說明                       |
| -----: | ------------------------ | ------------------------ |
|      1 | `unit`                   | Engine unit ID           |
|      2 | `cycle`                  | Operational cycle        |
|      3 | `op_setting_1`           | Operational setting 1    |
|      4 | `op_setting_2`           | Operational setting 2    |
|      5 | `op_setting_3`           | Operational setting 3    |
|   6–26 | `sensor_1` ~ `sensor_21` | 21 個 sensor measurements |

因此資料結構可以整理為：

```text
unit
cycle
operational settings: 3 columns
sensor measurements: 21 columns
```

總共：

```text
2 + 3 + 21 = 26 columns
```

---

## 7. Sensor Measurements

C-MAPSSData 中共有 21 個 sensor measurements。

常見命名方式如下：

| Sensor    | Symbol    | Description                  |
| --------- | --------- | ---------------------------- |
| sensor_1  | T2        | Fan inlet total temperature  |
| sensor_2  | T24       | LPC outlet total temperature |
| sensor_3  | T30       | HPC outlet total temperature |
| sensor_4  | T50       | LPT outlet total temperature |
| sensor_5  | P2        | Fan inlet pressure           |
| sensor_6  | P15       | Bypass-duct total pressure   |
| sensor_7  | P30       | HPC outlet total pressure    |
| sensor_8  | Nf        | Physical fan speed           |
| sensor_9  | Nc        | Physical core speed          |
| sensor_10 | epr       | Engine pressure ratio        |
| sensor_11 | Ps30      | HPC outlet static pressure   |
| sensor_12 | phi       | Fuel flow / Ps30 ratio       |
| sensor_13 | NRf       | Corrected fan speed          |
| sensor_14 | NRc       | Corrected core speed         |
| sensor_15 | BPR       | Bypass ratio                 |
| sensor_16 | farB      | Burner fuel-air ratio        |
| sensor_17 | htBleed   | Bleed enthalpy               |
| sensor_18 | Nf_dmd    | Demanded fan speed           |
| sensor_19 | PCNfR_dmd | Demanded corrected fan speed |
| sensor_20 | W31       | HPT coolant bleed            |
| sensor_21 | W32       | LPT coolant bleed            |

---

## 8. Train Data 的 RUL Label 建立方式

`train_FDxxx.txt` 沒有直接提供 RUL column。

因為 training data 中每一台 engine 都會一路記錄到 failure，所以可以根據該 engine 的最後一個 cycle 自行建立 RUL label。

公式如下：

```text
RUL = max_cycle_of_unit - current_cycle
```

例如某台 engine 最後在 cycle 200 failure：

| unit | cycle | RUL |
| ---: | ----: | --: |
|    1 |     1 | 199 |
|    1 |     2 | 198 |
|    1 |   100 | 100 |
|    1 |   199 |   1 |
|    1 |   200 |   0 |

也就是說，越接近 failure，RUL 越小。

---

## 9. Test Data 的 RUL Label 建立方式

`test_FDxxx.txt` 中的 engine sequence 不會記錄到 failure，而是會在 failure 前某個時間點停止。

`RUL_FDxxx.txt` 提供的是：

```text
每台 test engine 在最後一個觀測 cycle 之後，還剩下多少 cycles 會 failure
```

若要為 test data 的每個 row 建立 RUL，可以使用：

```text
RUL_at_cycle_t = max_cycle_in_test_unit - current_cycle + true_RUL_after_last_cycle
```

也就是：

```text
RUL = last_observed_cycle - current_cycle + RUL_file_value
```

其中：

| 變數                    | 說明                                     |
| --------------------- | -------------------------------------- |
| `last_observed_cycle` | 該 test engine 在 test data 中最後出現的 cycle |
| `current_cycle`       | 目前 row 的 cycle                         |
| `RUL_file_value`      | `RUL_FDxxx.txt` 中對應 engine 的真實剩餘壽命     |

---

## 10. 建模輸入與輸出

### Input

通常模型輸入包含：

```text
op_setting_1
op_setting_2
op_setting_3
sensor_1
sensor_2
...
sensor_21
```

也就是 24 個 features：

```text
3 operational settings + 21 sensor measurements
```

有些方法會只使用 sensor measurements，有些方法會保留 operational settings。

### Output

模型輸出通常是：

```text
RUL
```

這是一個 regression problem。

因此模型形式可以表示為：

```text
X: historical sensor sequence
y: Remaining Useful Life
```

---

## 11. 常見前處理方式

### 11.1 Feature Normalization

因為不同 sensor 的數值尺度差異很大，通常需要 normalization。

常見方法：

```text
StandardScaler
MinMaxScaler
Z-score normalization
```

### 11.2 Condition-wise Normalization

FD002 與 FD004 有 6 種 operating conditions。

這代表 sensor 數值變化不只受到 degradation 影響，也會受到操作條件影響。

因此 FD002 / FD004 常見做法是：

```text
根據 operational settings 分群
再針對不同 operating condition 做 normalization
```

或是：

```text
將 operational settings 作為模型輸入
讓模型自行學習不同操作條件下的 sensor pattern
```

### 11.3 RUL Clipping

很多研究會對 RUL 做上限截斷，例如：

```text
RUL = min(RUL, 125)
```

原因是 engine 在早期健康狀態下，sensor pattern 變化不明顯，精確區分 RUL 250 與 RUL 300 的意義不大。

RUL clipping 可以讓模型更專注於後期 degradation pattern。

---

## 12. 適合的模型方法

此 dataset 常用於時間序列 regression，因此常見方法包括：

### Machine Learning

```text
Linear Regression
Random Forest
XGBoost
LightGBM
SVR
```

### Deep Learning

```text
MLP
1D CNN
LSTM
GRU
BiLSTM
TCN
Transformer
Autoencoder-based methods
```

### Predictive Maintenance / PHM 方法

```text
Health Index Learning
Degradation Trend Modeling
Sequence-to-One RUL Prediction
Sequence-to-Sequence RUL Prediction
Uncertainty-aware RUL Prediction
```

---

## 13. Dataset 難度比較

| Dataset | 難度 | 原因                                |
| ------- | -- | --------------------------------- |
| FD001   | 低  | 單一操作條件、單一 fault mode              |
| FD002   | 中高 | 多操作條件造成 sensor distribution shift |
| FD003   | 中  | 多 fault modes，但操作條件固定             |
| FD004   | 高  | 同時包含多操作條件與多 fault modes           |

一般來說：

```text
FD001 < FD003 < FD002 < FD004
```

FD004 通常是最具挑戰性的子資料集。

---

## 14. Dataset 使用情境

C-MAPSSData 可以用於模擬下列問題：

```text
目前這台引擎還能運轉多久？
哪一台 engine 最需要維修？
sensor pattern 是否出現異常？
能否提前預測 failure？
如何根據 degradation trend 安排 maintenance？
```

因此它非常適合用於：

```text
Predictive Maintenance
Prognostics
Industrial AI
Time-series Forecasting
Anomaly Detection
Remaining Useful Life Estimation
```

---

## 15. 總結

NASA C-MAPSSData 是一個經典的 Remaining Useful Life 預測資料集。

它的資料形式是多台 turbofan engine 的 multivariate time series，每個 row 代表一台 engine 在某個 cycle 的 operational settings 與 sensor measurements。

Training data 提供完整 run-to-failure sequence，test data 則在 failure 前截斷，並透過 `RUL_FDxxx.txt` 提供 test engine 的真實剩餘壽命。

此 dataset 的主要挑戰包括：

```text
sensor noise
不同 engine 的 initial wear 差異
不同 operating conditions
不同 fault modes
degradation pattern 不明顯
長時間序列建模
RUL regression
```

其中 FD001 最適合入門，FD004 最適合測試模型在複雜操作條件與多故障模式下的泛化能力。