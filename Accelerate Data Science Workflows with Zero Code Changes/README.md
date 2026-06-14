# NVIDIA DLI：Accelerate Data Science Workflows with Zero Code Changes 輔助教學教材

本教材用來輔助說明 4 個 notebook 的內容與教學重點：

1. `00_introduction.ipynb`
2. `01_cuDF.ipynb`
3. `02_cuML.ipynb`
4. `03_cuGraph.ipynb`

這組教材的核心目標是讓學員理解：在資料科學工作流程中，如何透過 RAPIDS 生態系，在盡量不改動原本 Python / pandas / scikit-learn / NetworkX 程式碼的前提下，利用 GPU 加速資料處理、機器學習與圖分析任務。

---

## 一、課程整體定位

### 課程主題

**Accelerate Data Science Workflows with Zero Code Changes** 的主軸是「以最小程式碼改動取得 GPU 加速」。課程透過 NVIDIA RAPIDS 的三個主要方向進行示範：

| Notebook | 主題 | 對應工具 | 核心概念 |
|---|---|---|---|
| `00_introduction.ipynb` | 課程與環境介紹 | JupyterLab | 熟悉 notebook 操作與 lab 環境 |
| `01_cuDF.ipynb` | GPU 加速 pandas workflow | cuDF / `cudf.pandas` | 讓既有 pandas 程式碼盡量零改動跑在 GPU 上 |
| `02_cuML.ipynb` | CPU/GPU 可切換的機器學習流程 | cuML | 以類 scikit-learn API 在 CPU/GPU 間切換執行 |
| `03_cuGraph.ipynb` | GPU 加速 NetworkX 圖分析 | cuGraph / `nx-cugraph` | 透過 NetworkX backend 機制加速圖演算法 |

### 建議教學順序

建議依照檔名前綴順序授課：

1. 先用 `00_introduction.ipynb` 確認學員能操作 JupyterLab。
2. 接著用 `01_cuDF.ipynb` 建立「zero code changes」最直觀的體驗：原本 pandas 程式碼載入 `cudf.pandas` 後即可加速。
3. 再用 `02_cuML.ipynb` 說明機器學習流程中，如何管理 CPU/GPU 執行平台。
4. 最後用 `03_cuGraph.ipynb` 展示圖分析任務在大型資料上使用 GPU 的優勢。

---

## 二、`00_introduction.ipynb`：課程與 JupyterLab 操作介紹

### 1. 檔案定位

`00_introduction.ipynb` 是整門課的入口。這個 notebook 不是在教特定 RAPIDS 函式庫，而是讓學員熟悉課程目標、JupyterLab 介面與 notebook 操作方式。

### 2. 學習目標

完成這份 notebook 後，學員應該能夠：

- 理解本課程要解決的問題：在 CPU/GPU 之間建立一致的資料科學 workflow。
- 知道 RAPIDS 在課程中的角色。
- 熟悉 JupyterLab 的基本介面。
- 執行 notebook cell。
- 在 notebook 中執行簡單的 Python 程式與 terminal command。

### 3. 主要內容

#### 課程總覽

notebook 一開始說明本課程會使用 NVIDIA RAPIDS 開源軟體套件，讓資料處理、機器學習與圖分析工作可以在 GPU 上加速。

課程拆成三個主要技術單元：

1. **cuDF**：加速 pandas workflow。
2. **cuML**：在 CPU 與 GPU 上執行類 scikit-learn 的機器學習 workflow。
3. **cuGraph**：透過 `nx-cugraph` backend 加速 NetworkX 圖分析。

#### JupyterLab 操作

這個部分介紹 JupyterLab 的基本元件：

- 左側 file browser。
- 中央 main work area。
- notebook cell。
- `Shift + Enter` 執行 cell。
- 使用 `!` 在 notebook cell 中執行 terminal command。
- 使用 kernel interrupt 停止執行中的程式。

#### 練習內容

notebook 提供兩個入門練習：

```python
print('This is just a simple print statement')
```

以及：

```bash
!echo 'This is another simple print statement'
```

第一個練習用來確認 Python code cell 可以執行；第二個練習用來確認學員理解 `!` 可以呼叫 shell command。

### 4. 教學提示

講師可以在這份 notebook 補充：

- notebook cell 分成 markdown cell 與 code cell。
- `Shift + Enter` 會執行目前 cell 並移到下一個 cell。
- 若程式卡住，可以使用上方工具列的 stop button 或 `Kernel > Interrupt Kernel`。
- notebook 中的 `!command` 是在作業系統 shell 層級執行，不是 Python 語法。

### 5. 適合提問

- notebook 和一般 `.py` 檔案有什麼差異？
- 為什麼資料科學課程常用 JupyterLab？
- 在 notebook 中執行 shell command 的好處是什麼？

---

## 三、`01_cuDF.ipynb`：使用 `cudf.pandas` 加速 pandas workflow

### 1. 檔案定位

`01_cuDF.ipynb` 是本課程最能體現「zero code changes」概念的 notebook。它先使用標準 pandas 分析資料，再重新載入 `cudf.pandas`，觀察相同 pandas 程式碼在 GPU 加速下的執行差異。

### 2. 學習目標

完成這份 notebook 後，學員應該能夠：

- 理解 cuDF 是 GPU DataFrame library。
- 理解 `cudf.pandas` 的 pandas accelerator mode。
- 使用 pandas 讀取與分析 Parquet 資料。
- 使用 `%load_ext cudf.pandas` 啟用 pandas 加速模式。
- 比較標準 pandas 與 GPU 加速後的執行時間。
- 使用 profiling 工具觀察哪些操作跑在 GPU，哪些 fallback 到 CPU。
- 理解第三方函式庫與 `cudf.pandas` proxy object 的互動方式。

### 3. 使用資料

notebook 使用的是 NYC Open Data 的 **Parking Violations Issued - Fiscal Year 2022** 資料集。為了加快下載速度，notebook 透過 NVIDIA 提供的資料來源下載 Parquet 檔案：

```bash
!wget https://data.rapids.ai/datasets/nyc_parking/nyc_parking_violations_2022.parquet
```

在課程中可以提醒學員：這是實務上常見的 tabular data 分析情境，資料量比玩具資料集大，更容易看出 GPU 加速差異。

### 4. 主要流程

#### Step 1：確認 GPU 與 cuDF 環境

notebook 先確認環境中有 NVIDIA GPU：

```bash
!nvidia-smi
```

接著確認 `cudf` 可以被正確匯入：

```python
import cudf
```

#### Step 2：安裝視覺化套件

notebook 安裝 `plotly-express`，後續用來視覺化不同州的 pickup truck 比例：

```bash
!pip install plotly-express
```

#### Step 3：使用標準 pandas 分析資料

notebook 先用 pandas 讀取部分欄位：

```python
import pandas as pd

# read 5 columns data:
df = pd.read_parquet(
    "nyc_parking_violations_2022.parquet",
    columns=[
        "Registration State",
        "Violation Description",
        "Vehicle Body Type",
        "Issue Date",
        "Summons Number",
    ],
)
```

接著回答三個資料分析問題：

1. **不同州車輛最常見的違規類型是什麼？**
2. **哪些車體類型最常出現在違規資料中？**
3. **一週中哪幾天違規數較多或較少？**

這些問題涵蓋常見 pandas 操作：

- `read_parquet`
- `value_counts`
- `groupby`
- `agg`
- `rename`
- `sort_values`
- datetime 轉換
- `.dt.weekday`
- method chaining

#### Step 4：用 `%%time` 比較執行時間

notebook 使用 Jupyter magic command `%%time` 測量資料讀取與分析所需時間。這一步是為了建立 baseline，也就是「沒有 GPU 加速前」的 pandas 執行時間。

#### Step 5：啟用 `cudf.pandas`

notebook 會重新啟動 kernel，模擬在 notebook 開頭就載入 `cudf.pandas` 的正確使用方式：

```python
get_ipython().kernel.do_shutdown(restart=True)
```

接著啟用 pandas accelerator mode：

```python
%load_ext cudf.pandas
```

啟用後，原本 `import pandas as pd` 的程式碼可以保留，但 pandas 的 DataFrame / Series 會被代理成能夠在 cuDF 與 pandas 之間 dispatch 的 proxy object。

#### Step 6：重新執行相同 pandas 程式碼

啟用 `cudf.pandas` 後，notebook 重新執行先前相同的 pandas 分析流程，並再次測量執行時間。這段是本 notebook 的教學重點：

> 程式碼幾乎不需要修改，但支援的操作可以改由 GPU 執行，進而縮短執行時間。

#### Step 7：理解效能與 fallback

notebook 介紹兩個 profiling 工具：

```python
%%cudf.pandas.profile
```

以及：

```python
%%cudf.pandas.line_profile
```

這兩個工具可以協助學員判斷：

- 哪些 pandas 操作被 cuDF 加速。
- 哪些操作 fallback 到 CPU pandas。
- 哪些步驟可能造成 CPU/GPU 資料搬移成本。

notebook 也示範：

```python
df.count(axis=0)
```

通常可由 cuDF 加速；但：

```python
df.count(axis=1)
```

因為特定參數支援度問題，可能 fallback 到 pandas，因此較慢。

#### Step 8：與第三方函式庫整合

notebook 使用 `plotly.express` 視覺化各州 pickup truck 的比例。這個段落的重點不是 Plotly 本身，而是說明：

- `cudf.pandas` 的 proxy object 可以傳入接受 pandas 物件的第三方函式庫。
- 第三方函式庫內部若使用 pandas 操作，也可能受益於 `cudf.pandas` 的加速。

### 5. 教學重點整理

| 主題 | 教學重點 |
|---|---|
| pandas baseline | 先讓學員看到標準 pandas 的寫法與執行時間 |
| `cudf.pandas` | 使用 `%load_ext cudf.pandas` 啟用加速模式 |
| zero code changes | 重跑相同 pandas 程式碼，比較時間差異 |
| proxy object | pandas API 背後被代理到 cuDF 或 pandas |
| fallback | cuDF 不支援的操作會回到 CPU pandas |
| profiling | 用 profile 找出 GPU/CPU 執行分布 |
| third-party libraries | 測試與 Plotly 等套件的整合情境 |

### 6. 常見學員疑問

#### Q1：是不是所有 pandas 程式碼都會變快？

不一定。只有 cuDF 支援的操作才會在 GPU 上加速。不支援的操作會 fallback 到 pandas，可能因為 CPU/GPU 資料搬移而變慢。

#### Q2：為什麼小資料不一定比較快？

GPU 有資料搬移與啟動 overhead。若資料很小，CPU 執行可能已經足夠快，GPU 加速優勢不明顯。

#### Q3：為什麼要先 restart kernel 再 `%load_ext cudf.pandas`？

一般建議在 notebook 一開始、匯入 pandas 之前啟用 `cudf.pandas`。這樣 pandas API 才能正確被 proxy 機制接管。

### 7. 延伸練習

- 請學員把 method chaining 拆成多個中間變數，觀察每一步的結果。
- 修改分析問題，例如找出各州違規數排名前 5 的車體類型。
- 使用 `%%cudf.pandas.line_profile` 找出 fallback 到 CPU 的程式碼。
- 嘗試使用更小的 sample，比較 GPU 加速是否仍然明顯。

---

## 四、`02_cuML.ipynb`：cuML 的 CPU/GPU 執行切換與模型序列化

### 1. 檔案定位

`02_cuML.ipynb` 介紹 cuML 在機器學習流程中的 CPU/GPU 統一執行能力。這份 notebook 的重點不是訓練複雜模型，而是示範如何用相近於 scikit-learn 的 API，在不同硬體環境中重用同一份程式碼。

### 2. 學習目標

完成這份 notebook 後，學員應該能夠：

- 理解 cuML 是 RAPIDS 中負責機器學習演算法的套件。
- 知道 cuML 提供類似 scikit-learn 的使用方式。
- 理解 `cuml-cpu` 與完整 cuML 套件的使用情境。
- 使用 `using_device_type()` 指定某段程式在 CPU 或 GPU 上執行。
- 使用 `set_global_device_type()` 設定全域預設執行裝置。
- 理解部分 estimator 可以跨 CPU/GPU 訓練、序列化與推論。

### 3. 主要概念

#### cuML 的角色

cuML 提供 GPU 加速的機器學習演算法，API 設計接近 scikit-learn。對資料科學使用者而言，學習成本較低，因為許多 estimator 的使用方式與 familiar 的 scikit-learn pattern 類似：

```python
model = Estimator()
model.fit(X_train, y_train)
predictions = model.predict(X_test)
```

#### CPU/GPU 統一 workflow

notebook 說明 cuML 從特定版本開始提供 CPU 與 GPU 執行能力，目標是讓使用者可以：

- 在沒有 GPU 的開發環境先做 prototyping。
- 在有 GPU 的 production server 上加速執行。
- 在部分 estimator 上進行跨裝置訓練與推論。
- 減少整合其他 PyData 生態系工具時的 dispatch boilerplate code。

> 授課提醒：notebook 內提到的版本資訊屬於教材撰寫時的背景，實際授課時應以當天 lab 環境安裝版本為準。

### 4. 主要流程

#### Step 1：使用 CPU package 直接執行 cuML 程式

notebook 以 Iris dataset 與 UMAP 為例，示範同一段 `import cuml` 的程式碼可以在 GPU 系統或 CPU-only 系統執行。

核心流程如下：

```python
import cuml
import pandas as pd

from cuml.manifold.umap import UMAP
from sklearn import datasets
from sklearn.model_selection import train_test_split
from sklearn.manifold import trustworthiness

iris = datasets.load_iris()
iris_df = pd.DataFrame(iris.data, columns=iris.feature_names)

embedding = UMAP(
    n_neighbors=10,
    min_dist=0.01,
    init="random",
).fit_transform(iris_df)

trust = trustworthiness(iris_df, embedding)
print(trust)
```

這段可用來說明：cuML 的 estimator interface 與 scikit-learn 的資料科學 workflow 很接近。

#### Step 2：準備 synthetic datasets

notebook 接著建立兩種資料：

- `make_blobs`：用於 nearest neighbors 範例。
- `make_regression`：用於 linear regression 與模型序列化範例。

這部分可以提醒學員：

- clustering / nearest neighbors 常用於非監督或相似度搜尋情境。
- regression dataset 則用來示範 supervised learning 的訓練與推論。

#### Step 3：使用 `using_device_type()` 控制局部執行裝置

notebook 示範：即使在完整 cuML 環境中，也可以用 context manager 指定某段程式在 CPU 執行。

```python
from cuml.neighbors import NearestNeighbors
from cuml.common.device_selection import using_device_type

nn = NearestNeighbors()
with using_device_type('cpu'):
    nn.fit(X_train_blobs)
    nearest_neighbors = nn.kneighbors(X_test_blobs)
```

這段適合說明：

- 某些小資料或特定參數情境，在 CPU 上執行可能更合適。
- 局部指定裝置可以避免把整個程式改成 CPU 或 GPU。
- 對 library integration 而言，context manager 是很方便的控制方式。

#### Step 4：使用 `set_global_device_type()` 設定全域預設裝置

notebook 接著示範全域裝置設定：

```python
from cuml.common.device_selection import set_global_device_type, get_global_device_type

initial_device_type = get_global_device_type()
print('default execution device:', initial_device_type)

set_global_device_type('cpu')
print('new device type:', get_global_device_type())
```

這段適合說明：

- 預設情況下，cuML 會傾向使用 GPU/device。
- 在共享 GPU 的環境中，有時可能希望把某些任務預設放到 CPU。
- 全域設定會影響後續 estimator 的預設執行平台。

#### Step 5：跨裝置訓練、序列化與推論

notebook 使用 `LinearRegression` 搭配 `pickle` 示範模型儲存與載入：

```python
import pickle
from cuml.linear_model import LinearRegression

lin_reg = LinearRegression()
lin_reg.fit(X_train_reg, y_train_reg)

pickle.dump(lin_reg, open("lin_reg.pkl", "wb"))
del lin_reg
```

接著載入模型並推論：

```python
recovered_lin_reg = pickle.load(open("lin_reg.pkl", "rb"))
predictions = recovered_lin_reg.predict(X_test_reg)
print(predictions[0:10])
```

這段的教學重點是：部分 estimator 支援在一種裝置上訓練後，序列化並在另一種裝置上載入與推論。

### 5. 教學重點整理

| 主題 | 教學重點 |
|---|---|
| cuML | GPU 加速機器學習演算法，API 類似 scikit-learn |
| `cuml-cpu` | 可在 CPU-only 環境使用部分 cuML estimator |
| UMAP 範例 | 用 Iris dataset 示範低維嵌入與 trustworthiness 評估 |
| `using_device_type()` | 局部指定 CPU/GPU 執行 |
| `set_global_device_type()` | 設定全域預設執行平台 |
| serialization | 用 `pickle` 儲存模型並重新載入推論 |

### 6. 常見學員疑問

#### Q1：cuML 是否等同於 scikit-learn？

不是。cuML 的 API 設計接近 scikit-learn，但底層實作與支援演算法不同。可以把它理解成 RAPIDS 生態系中針對 GPU 加速最佳化的機器學習工具。

#### Q2：什麼時候要用 CPU？什麼時候要用 GPU？

若資料量小、GPU 正被其他任務占用，或特定參數只支援 CPU，使用 CPU 可能更合理。若資料量大或演算法平行化程度高，GPU 通常更有優勢。

#### Q3：所有 estimator 都能跨 CPU/GPU 序列化嗎？

不是。只有部分 estimator 支援跨裝置訓練、序列化與推論。授課時可以引導學員查看 notebook 中的支援表格與實際環境文件。

### 7. 延伸練習

- 修改 UMAP 的 `n_neighbors` 或 `min_dist`，觀察 embedding 與 trustworthiness 是否改變。
- 將 `using_device_type('cpu')` 改為 `using_device_type('gpu')`，觀察執行差異。
- 嘗試在不同全域設定下建立 estimator，確認預設裝置是否改變。
- 比較 `pickle` 儲存前後模型推論結果是否一致。

---

## 五、`03_cuGraph.ipynb`：使用 `nx-cugraph` 加速 NetworkX 圖分析

### 1. 檔案定位

`03_cuGraph.ipynb` 介紹如何使用 `nx-cugraph` 作為 NetworkX backend，在保留 NetworkX API 的情況下，將支援的圖演算法交給 RAPIDS cuGraph 在 GPU 上執行。

這份 notebook 的核心訊息是：

> 對 NetworkX 使用者而言，可以透過 backend 機制，在少量或零程式碼改動的情況下，讓大型圖分析任務取得 GPU 加速。

### 2. 學習目標

完成這份 notebook 後，學員應該能夠：

- 理解 cuGraph 與 `nx-cugraph` 的角色。
- 知道 NetworkX backend dispatch 的基本概念。
- 使用 `NETWORKX_AUTOMATIC_BACKENDS` 啟用自動 backend dispatch。
- 使用 `backend="cugraph"` 明確指定某次 NetworkX API 呼叫使用 cuGraph backend。
- 比較小圖與大圖在 GPU 加速下的差異。
- 理解 type-based dispatching 的使用方式。
- 理解 `betweenness_centrality` 在圖分析中的代表性意義。

### 3. 背景概念

#### NetworkX

NetworkX 是 Python 常用的圖分析套件，適合建立圖、操作節點與邊、計算 centrality、community detection 等圖演算法。

#### cuGraph

cuGraph 是 RAPIDS 中負責 GPU 加速圖分析的套件，適合處理大規模 graph workload。

#### `nx-cugraph`

`nx-cugraph` 是 NetworkX 的 backend，可讓 NetworkX 在支援的演算法上把計算派發到 cuGraph。這使得使用者可以保留熟悉的 NetworkX API，同時利用 GPU 加速。

### 4. 主要流程

#### Step 1：定義 helper function

notebook 先定義 `reimport_networkx()`，用來在示範過程中重新匯入 NetworkX。這是因為 NetworkX 會在 import time 讀取 backend 設定，因此修改環境變數後需要重新 import 才能模擬 shell 中設定 backend 的效果。

#### Step 2：使用 NetworkX 計算 Karate Club graph 的 betweenness centrality

notebook 使用 Zachary's Karate Club graph：

```python
import networkx as nx
karate_club_graph = nx.karate_club_graph()
```

這是一個小型圖資料集，包含 34 個 nodes 與 78 條 edges，常用於圖分析教學。

接著使用 NetworkX 預設實作計算：

```python
karate_nx_bc_results = nx.betweenness_centrality(karate_club_graph)
```

這段可以用來介紹 betweenness centrality：

- 衡量節點位於其他節點最短路徑上的程度。
- 分數越高，通常代表節點在資訊傳遞或網路連通中越關鍵。

#### Step 3：啟用自動 GPU backend

notebook 示範透過環境變數指定 NetworkX 自動使用 `cugraph` backend：

```python
import os
os.environ["NETWORKX_AUTOMATIC_BACKENDS"] = "cugraph"
nx = reimport_networkx()
karate_club_graph = nx.karate_club_graph()
```

之後重新執行：

```python
karate_cg_bc_results = nx.betweenness_centrality(karate_club_graph)
```

這裡的重點是：程式碼呼叫仍然是 NetworkX API，但支援的部分可由 `nx-cugraph` backend 執行。

#### Step 4：比較小圖結果與效能

notebook 指出：在 Karate Club 這種小型圖上，GPU 版本可能反而比較慢，因為：

- GPU 啟動與資料搬移有 overhead。
- 小圖計算量太低，無法抵銷 overhead。

這是一個重要觀念：GPU 加速不是任何情況都更快，必須看資料規模與運算型態。

#### Step 5：使用大型 U.S. Patent Citation Network

接著 notebook 使用 U.S. Patent Citation Network，資料規模約為數百萬 nodes 與一千多萬 edges。這個段落用來展示 GPU 對大型圖的加速優勢。

notebook 先用 NetworkX 預設實作執行：

```python
results = nx.betweenness_centrality(cit_patents_graph, k=k)
```

由於大型圖完整計算成本很高，notebook 使用 `k` 參數控制 approximation 的 sample 數量。

#### Step 6：明確指定 `backend="cugraph"`

notebook 接著示範不透過環境變數，而是在函式呼叫時直接指定 backend：

```python
nx.betweenness_centrality(
    cit_patents_graph,
    k=k,
    backend="cugraph",
)
```

這段是實務上很重要的寫法，因為它可以對單次演算法呼叫指定 backend，不一定要改變整個程式的全域設定。

#### Step 7：type-based dispatching

最後 notebook 介紹 type-based dispatching。做法是直接匯入 `nx_cugraph`，並把 NetworkX graph 轉換成 `nx-cugraph` graph：

```python
import nx_cugraph as nxcg

nxcg_cit_patents_graph = nxcg.from_networkx(cit_patents_graph)
```

之後只要傳入 `nx-cugraph` graph object，NetworkX dispatcher 就會依照物件型別自動使用對應 backend。

這種方式的優點是：

- 可以避免每次呼叫時重複把資料從 CPU graph 轉到 GPU graph。
- 對大量重複圖演算法呼叫更有效率。
- 行為更明確，較不依賴外部環境變數。

### 5. 教學重點整理

| 主題 | 教學重點 |
|---|---|
| NetworkX backend | NetworkX 可透過 backend dispatch 使用其他實作 |
| `nx-cugraph` | 讓支援的 NetworkX 演算法交給 cuGraph 在 GPU 執行 |
| 小圖效能 | 小資料可能因 GPU overhead 而不會比較快 |
| 大圖效能 | 大型 graph workload 通常更能展現 GPU 平行化優勢 |
| `backend="cugraph"` | 對單次呼叫明確指定 cuGraph backend |
| type-based dispatch | 透過 graph object 型別決定 backend，減少重複轉換成本 |
| `k` 參數 | 控制 betweenness centrality approximation 的 sample 數量 |

### 6. 常見學員疑問

#### Q1：為什麼 Karate Club graph 用 GPU 反而慢？

因為圖太小。GPU 的資料搬移、初始化與 dispatch overhead 可能比實際計算還大。大型圖才更容易展現 GPU 加速效果。

#### Q2：`NETWORKX_AUTOMATIC_BACKENDS` 和 `backend="cugraph"` 差在哪？

`NETWORKX_AUTOMATIC_BACKENDS` 是環境層級設定，會讓 NetworkX 在可支援時自動使用指定 backend。`backend="cugraph"` 則是單次函式呼叫層級的明確指定。

#### Q3：type-based dispatching 什麼時候有用？

當同一個大型圖會被重複執行多個演算法時，先轉成 `nx-cugraph` graph 可以避免每次都重複轉換資料，有助於降低 overhead。

### 7. 延伸練習

- 比較 `k=40` 與 `k=150` 在 GPU 上的執行時間與結果差異。
- 將 `backend="cugraph"` 移除，觀察 NetworkX 預設實作的時間。
- 討論小圖、中文字詞網路、社群網路、交通路網等不同 graph size 對 GPU 加速的影響。
- 嘗試查詢 `nx-cugraph` 支援哪些 NetworkX 演算法，並判斷哪些適合課堂展示。

---

## 六、四個 notebook 的教學串接方式

### 1. 從「操作環境」到「資料科學 workflow」

`00_introduction.ipynb` 先確保所有學員能執行 notebook。這一步雖然技術門檻低，但對 hands-on lab 非常重要。若學員不熟悉 JupyterLab，後續 cuDF、cuML、cuGraph 的操作都會受到影響。

### 2. 用 cuDF 建立 GPU 加速直覺

`01_cuDF.ipynb` 最適合用來建立第一個「GPU 加速真的有效」的體驗。因為 pandas 是多數資料科學學員熟悉的工具，使用相同 pandas code 對比標準 pandas 與 `cudf.pandas`，可以很直觀地說明 zero code changes 的價值。

### 3. 用 cuML 說明機器學習 workflow 的部署彈性

`02_cuML.ipynb` 的重點不只是速度，而是彈性：同一份 ML workflow 可以依照硬體條件在 CPU 或 GPU 上執行。這對 prototyping、共享 GPU、production deployment 都很有價值。

### 4. 用 cuGraph 展示大型圖分析的加速場景

`03_cuGraph.ipynb` 可以放在最後，因為它需要學員理解 backend dispatch 與 GPU overhead。這份 notebook 特別適合強調：GPU 加速不是單純按下開關，而是要理解 workload 的資料規模與演算法特性。

---

## 七、建議課堂講解節奏

| 階段 | Notebook | 建議時間 | 教學重點 |
|---|---|---:|---|
| 環境熟悉 | `00_introduction.ipynb` | 10–15 分鐘 | JupyterLab、cell 執行、shell command |
| 資料處理加速 | `01_cuDF.ipynb` | 45–60 分鐘 | pandas baseline、`cudf.pandas`、profiling、fallback |
| 機器學習 workflow | `02_cuML.ipynb` | 30–45 分鐘 | CPU/GPU 切換、device context、模型序列化 |
| 圖分析加速 | `03_cuGraph.ipynb` | 40–60 分鐘 | NetworkX backend、betweenness centrality、大圖加速 |

---

## 八、講師可強調的共同觀念

### 1. Zero code changes 不代表 zero understanding

雖然 RAPIDS 提供許多「少改或不改程式碼」的加速方式，但使用者仍需理解：

- 哪些操作支援 GPU。
- 哪些操作會 fallback 到 CPU。
- 資料在 CPU/GPU 間搬移會產生成本。
- 小資料不一定適合 GPU。
- 大資料與高度平行化任務更容易受益於 GPU。

### 2. GPU 加速要看 workload

這四份 notebook 剛好能示範三種 workload：

| Workload | 對應 notebook | GPU 加速特性 |
|---|---|---|
| Tabular data processing | `01_cuDF.ipynb` | 大量 DataFrame aggregation / filtering / grouping 易受益 |
| Machine learning | `02_cuML.ipynb` | estimator 與資料規模會影響 CPU/GPU 適用性 |
| Graph analytics | `03_cuGraph.ipynb` | 大型 graph 與高平行化演算法較能展現優勢 |

### 3. Profiling 是必要步驟

不要只看「有沒有開 GPU」，更要看實際程式碼是否真的跑在 GPU 上。`01_cuDF.ipynb` 的 profile 工具是很好的切入點，可以引導學員建立 performance diagnosis 的觀念。

### 4. API 相容性降低學習成本

RAPIDS 生態系的一大特點，是盡量讓使用者沿用熟悉的 API：

- pandas → `cudf.pandas`
- scikit-learn-like workflow → cuML
- NetworkX → `nx-cugraph`

這讓學員不需要完全重寫既有資料科學流程，就能逐步導入 GPU 加速。

---

## 九、課後複習題

### 基礎題

1. `00_introduction.ipynb` 中，如何在 notebook cell 執行 shell command？
2. `01_cuDF.ipynb` 中，啟用 `cudf.pandas` 的 magic command 是什麼？
3. `02_cuML.ipynb` 中，哪個 context manager 可以局部指定 CPU 或 GPU 執行？
4. `03_cuGraph.ipynb` 中，如何在單次 NetworkX API 呼叫中指定使用 cuGraph backend？

### 理解題

1. 為什麼 `cudf.pandas` 不一定讓所有 pandas 程式碼變快？
2. 為什麼小型 graph 使用 GPU 可能比 CPU 慢？
3. `using_device_type()` 與 `set_global_device_type()` 的差異是什麼？
4. `NETWORKX_AUTOMATIC_BACKENDS` 與 `backend="cugraph"` 各適合什麼情境？

### 應用題

1. 若你有一份大型 CSV/Parquet 資料，想加速 groupby aggregation，會先嘗試哪個 notebook 的方法？
2. 若你正在開發機器學習模型，但開發機沒有 GPU、部署機有 GPU，`02_cuML.ipynb` 的哪個概念最有幫助？
3. 若你有一個大型社群網路圖，需要計算 centrality，`03_cuGraph.ipynb` 提供了哪些可行做法？

---

## 十、總結

這四個 notebook 共同展示了 RAPIDS 對資料科學 workflow 的三層加速方式：

1. **cuDF**：讓 pandas-style tabular data processing 能夠在 GPU 上加速。
2. **cuML**：讓機器學習 estimator 可以更彈性地在 CPU/GPU 間切換。
3. **cuGraph**：讓 NetworkX-style graph analytics 可以利用 GPU 處理大型圖資料。

## 補充

* [10 Minutes to cuDF and Dask cuDF](https://docs.rapids.ai/api/cudf/stable/user_guide/10min/)
