# 0_ALM_Database_Setup.ipynb

1. 下載config-reasoning_temp.yml，改名為config-reasoning.yml，放在config目錄
2. Optional: NASA C-MAPSSData 備份下載連結：https://www.kaggle.com/datasets/behrad3d/nasa-cmaps

# 1_ALM_Application_Setup.ipynb

1. 開啟終端機，輸入指令設定OpenAI API Key並啟動 NAT 後端服務
```bash
source /opt/predm-lab/bin/activate
export OPENAI_API_KEY="sk-proj-..."
nat serve --config_file=configs/config-reasoning.yml
```
2. client.check_health() 確認 NAT 後端服務正常運作
3. 使用client.query("PROMPT") 測試 NAT 後端是否正常回應
4. 開啟終端機，安裝NAT_Toolkit-UI 啟動前端介面
```bash
cd /dli/task/NAT_Toolkit-UI/ && npm install && npm run dev &
```
5. 使用產生的連結開啟前端介面，在左下角Settings 開啟以下設定
   - ✅ Enable Intermediate Steps
   - ✅ Expand Intermediate Steps by Default

# 2_ALM_Config_Understanding.ipynb
## Exercise 2.1: Temperature Parameter Configuration
1. 開啟`configs/config-reasoning.yml`
2. 在reasoning_llm 中新增temperature參數，設定為0~1之間的值，存檔
```yaml
reasoning_llm:
  _type: nim
  model_name: ...
  base_url: ...
  temperature: 0.1  # (try 0.1 ~ 1.0)
```
3. 重新啟動 NAT 後端服務，讓設定生效

## Exercise 2.2: Plotting Tool Enhancement
修改繪圖工具，讓他可以接收使用者指定的顏色參數

1. `plot_line_chart_tool.py` - 新增 color 參數
2. `plot_utils.py` - 實作顏色參數的處理邏輯
3. `config-reasoning.yml` - 更新prompt

以下是中文版本，`file path`、`class`、`function`、`parameter`、`tool name` 等專有名詞與程式碼保留不改。

## Tasks

### Step 1: 修改 plotting tool

#### 1a. 編輯 `plot_line_chart_tool.py`

開啟：`/dli/task/asset_lifecycle_management_agent/src/asset_lifecycle_management_agent/plotting/plot_line_chart_tool.py`

**Change 1 — 在 input schema 中加入 `color`**
找到 `PlotLineChartInputSchema` class：

```python
class PlotLineChartInputSchema(BaseModel):
    data_json_path: str = Field(...)
    x_axis_column: str = Field(...)
    y_axis_column: str = Field(...)
    plot_title: str = Field(...)
    color: str = Field(description="The color for the line chart", default="blue")  # 新增 color 欄位
```

**Change 2 — 在 response function 中接收 `color`**
找到 `_response_fn`：

```python
async def _response_fn(data_json_path: str, x_axis_column: str,
                       y_axis_column: str, plot_title: str,
                       color: str = "blue") -> str:  # 新增 color 參數
```

**Change 3 — 將 `color` 傳給 `create_line_chart`**
在 `_response_fn` 裡找到呼叫 `create_line_chart` 的地方：

```python
html_filepath, png_filepath = create_line_chart(
    output_dir=config.output_folder,
    data_json_path=data_json_path,
    x_col=x_axis_column,
    y_col=y_axis_column,
    title=plot_title,
    color=color  # 新增 color 參數
)
```

---

#### 1b. 編輯 `plot_utils.py`

開啟：`/dli/task/asset_lifecycle_management_agent/src/asset_lifecycle_management_agent/plotting/plot_utils.py`

**Change 4 — 讓 `create_line_chart` 接收 `color`**
找到 function signature：

```python
def create_line_chart(output_dir: str, data_json_path: str, x_col: str,
                      y_col: str, title: str,
                      color: str = "blue") -> Tuple[str, Optional[str]]:  # 新增 color 參數
```

**Change 5 — 使用傳入的 color**
在 Plotly trace 裡找到 `line=dict(...)` 和 `marker=dict(...)`：

```python
line=dict(color=color, width=3),        # 原本預設顏色：'#1f77b4' 改成 color 參數
marker=dict(size=6, color=color),       # 原本預設顏色：'#1f77b4' 改成 color 參數
```

---

### Step 2: 更新 config 裡的 system prompt

在 `configs/config-reasoning.yml` 中，找到 `data_analysis_assistant` 底下的 `system_prompt`，並在 **Analysis and Plotting Tools** 區塊中加入以下這一行：

```text
- plot_line_chart: ... Extract color preference from user prompt (e.g., "plot in blue", "use red color") and pass as color parameter.
```

意思是：當使用者 prompt 中有指定顏色，例如 `"plot in blue"` 或 `"use red color"`，agent 需要擷取這個 color preference，並把它作為 `color` parameter 傳給 `plot_line_chart`。

---

### Step 3: 重啟 Backend


## Exercise 2.3： 新增Action Recommendation Tool
可參考 `Exercise_Solutions/Exercise_3_Action_Recommendation`中的檔案
### Step 1: 新增 Action recommendation tool

#### 1a: 建立 Tool File

建立以下檔案結構。你可能需要先建立 `recommenders/` directory：

```text
asset_lifecycle_management_agent/src/asset_lifecycle_management_agent/recommenders/
├── __init__.py                      # empty file
└── action_recommendation_tool.py    # the new tool
```

這個 tool 會遵循你在其他 tools 中已經看過的標準 NAT pattern：

1. **Config class** — 繼承 `FunctionBaseConfig`，並設定 `name="action_recommendation_tool"`。需要一個欄位：`llm_name`，用來指定產生 recommendations 時要使用哪一個 LLM。

2. **Input schema** — 使用 Pydantic `BaseModel`，包含兩個欄位：

   * `analysis_result: str` — 作為 recommendations 依據的 analysis output
   * `context: str` — 額外 context，例如 engine unit、dataset 等

3. **Response function** — 使用 analysis result 和 context 建立 prompt，透過 `builder.get_llm(config.llm_name)` 呼叫 LLM，並將輸出格式化為帶有 urgency levels 的 numbered list recommendations，例如 `URGENT / HIGH / MEDIUM / LOW`。

4. **Registration** — 使用 `@register_function(config_type=..., framework_wrappers=[LLMFrameworkEnum.LANGCHAIN])` 進行裝飾。

可以參考現有 tools，例如 `predict_rul_tool.py`，來撰寫 boilerplate structure。

---

#### 1b: Register the Tool

在 `register.py` 中新增一行 import：

```python
from .recommenders import action_recommendation_tool
```

---

#### 1c: 更新 config file

在 `configs/config-reasoning.yml` 中，新增兩個部分：

**a) 在 `functions:` 底下加入 tool config：**

```yaml
  action_recommendation:
    _type: action_recommendation_tool
    llm_name: reasoning_llm
```

**b) 在 `data_analysis_assistant.tool_names` 底下，把它加入 list：**

```yaml
    tool_names: [
      sql_retriever,
      predict_rul,
      ...,
      action_recommendation   # ← ADD
    ]
```

**c) 可選：在 `data_analysis_assistant.system_prompt` 中加入 usage hint：**

```yaml
      6. **Action Recommendation Tool**
         - Use action_recommendation to provide actionable maintenance recommendations
           based on any analysis results.
         - Call this tool after completing analysis to give users specific next steps.
```

# 3_ALM_Observability.ipynb: Observability & Monitoring

啟動指令：

```bash
source /opt/predm-lab/bin/activate
phoenix serve
```

安裝指令(啟動失敗時)：

```bash
source /opt/predm-lab/bin/activate
uv pip install "arize-phoenix==13.0.3" "arize-phoenix-evals==2.13.0"
```

Step3: 在 `config-reasoning.yml` 中開啟 Phoenix Tracing：

```yaml
general:
  use_uvloop: true
  # --- EXERCISE 3.1: Uncomment the lines below to enable Phoenix tracing ---
  telemetry:
    tracing:
      phoenix:
        _type: phoenix
        endpoint: http://localhost:6006/v1/traces
        project: asset-lifecycle-management
```

Step4: 重新啟動 NAT 後端服務
Step5: 在UI中使用以下 prompt 觸發 tracing：

```text
What is the ground truth RUL of unit_number 20 in dataset FD001?
```
```text
Retrieve sensor data for unit 15 in FD001 training data, predict its remaining useful life, and plot the predicted RUL over time
```

# 4_ALM_Evaluation.ipynb：評估與品質控管

開啟新terminal，執行評測指令：
```bash
source /opt/predm-lab/bin/activate
export OPENAI_API_KEY="sk-proj-..."
nat eval --config_file configs/config-reasoning.yml --dataset /dli/task/eval_set_lab.json
```
# 5_ALM_AnomalyDetection.ipynb


```yaml
embedders:
  # Text embedding model for vector database operations
  vanna_embedder:
    _type: nim
    model_name: "nvidia/llama-nemotron-embed-1b-v2"
    base_url: ${NVIDIA_BASE_URL}
```

## Exercise 4.2: 建立Custom Evaluation Set

使用Custom Evaluation Set評估模型
```bash
nat eval --config_file configs/config-reasoning.yml --dataset /dli/task/asset_lifecycle_management_agent/eval_set_custom.json
```

# ALM_assessment.ipynb：課程評量

建立新工具 `plot_line_chart_with_ma_tool` 有以下功能

1. 從 JSON 檔案載入感測器資料（與現有的折線圖工具相同）
2. 將原始資料繪製成折線圖
3. 計算 y 軸欄位的**移動平均**
4. 在同一張圖上**疊加顯示移動平均線**，作為第二條線（紅色）
5. **儲存一個 JSON 檔案**，其中包含原始資料，以及一個名為 **`moving_average`** 的欄位

注意事項：
* 工具名稱必須是 `plot_line_chart_with_ma_tool`
* 輸出JSON檔案中的移動平均欄位名稱必須是 `moving_average`

## Scoring

### 評分標準

所有交付項目都需要放在 `Assessment/output/` 資料夾中，以供評分：

| Deliverable                       | 評分內容                                 |      分數 |
| --------------------------------- | ------------------------------------ | ------: |
| `plot_line_chart_with_ma_tool.py` | 你完成的工具原始碼                            |      50 |
| `moving_average_output.json`      | 包含原始資料與 `moving_average` 欄位的 JSON 輸出 |      30 |
| `traces.json`                     | Phoenix trace 原始匯出檔案（JSON）           |      10 |
| `trace_analysis.json`             | 你對 trace 的分析結果                       |      10 |
| **總分**                            |                                      | **100** |

### Service 檢查

1. NAT Backend (Optional)
2. Phoenix Tracing
3. Front-end UI (Web)
4. Phoenix Dashboard (Web)

### Step 1: 工具撰寫

#### 1a. 修改`Assessment/plot_line_chart_with_ma_tool.py`

| block | 要新增的內容                              | 提示                                                                      |
| ----- | ----------------------------------- | ----------------------------------------------------------------------- |
| 1     | 在 input schema: class PlotLineChartWithMAInputSchema 中加入 `window_size` 參數 | `window_size: int = Field(description="...", default=10)`               |
| 2     | _response_fn中，使用 pandas 計算移動平均                    | `df_sorted['moving_average'] = df_sorted[COLUMN_NAME].rolling(window=WINDOW_SIZE).mean()` |
| 3     | _response_fn中，為移動平均新增第二條 Plotly trace, color用紅色             | 依照上方原始資料 trace 的寫法                                                      |
| 4     | _response_fn中，儲存包含原始資料與移動平均資料的 JSON               | `output_df.to_json(json_path, orient='records', indent=2)`                   |

#### 1b. 將此工具連結至Agent

* 將`Assessment/plot_line_chart_with_ma_tool.py`複製到`asset_lifecycle_management_agent/src/asset_lifecycle_management_agent/plotting`
* 在`asset_lifecycle_management_agent/src/asset_lifecycle_management_agent/plotting/__init__.py`中加入import
* `asset_lifecycle_management_agent/src/asset_lifecycle_management_agent/register.py`中加入import，並確保工具被註冊
* 修改`configs/config-reasoning.yml`

#### 1c. 重新啟動 NAT

使用以下prompt測試
```text
Retrieve sensor_measurement_4 data for unit 50 in FD001 and plot it with a moving average using window size 15
```

*檢查成果*
* Agent 會擷取感測器資料、呼叫你的工具，並回傳 inline chart
* 圖表會顯示**原始資料（藍色）**，並疊加顯示**移動平均線（紅色）**
* 系統會在 `agent_workspace/` 中建立一個檔案：`moving_average_output.json`，其中包含以下 columns：`time_in_cycles`、`sensor_measurement_4`、`moving_average`
* **Phoenix Dashboard** 中會出現一筆 trace，顯示完整 workflow

### Step 2: 儲存Code and JSON Output

將完成的 tool 檔案與 JSON 輸出檔案複製到 `Assessment/output/` 資料夾中。

### Step 3: 輸出Phoenix Trace

輸出Phoenix traces.json 到`Assessment/output/`

### Step 4: 檢查Trace Export

### Step 5: 分析Trace

### Step 6: 最終驗證

### Step 7: 繳交

# Optional_ALM_Codegen.ipynb: Code Generation & Complex Queries

### Step 1: 檢查Sandbox環境是否正常運作

### Step 2: 更新config

1. 新增`llms`
```yaml
coding_llm:
    _type: openai
    model_name: "gpt-4.1-mini"
    api_key: ${OPENAI_API_KEY}
    base_url: "https://api.openai.com/v1"
```
2. 新增`functions`
```yaml
code_generation_assistant:
    _type: code_generation_assistant
    llm_name: coding_llm
    code_execution_tool: code_execution
    verbose: true
    sandbox_db_path: "/database/nasa_turbo.db"
    allowed_libraries: [
        "pandas", "numpy", "matplotlib", "seaborn",
        "sqlite3", "json", "math", "statistics",
        "os", "pathlib", "datetime", "re", "csv",
        "scipy", "sklearn"
    ]

code_execution:
    _type: code_execution
    uri: http://code_execution_sandbox:6000/execute
    sandbox_type: local
    max_output_characters: 2000
```
3. 綁定工具至 agent
```yaml
tool_names: [
    sql_retriever, predict_rul,
    plot_distribution, plot_line_chart, plot_comparison,
    anomaly_detection, plot_anomaly,
    code_generation_assistant   # new
]
```
4. 開始測試
---

**整體結構**

這個 workspace 是一個「課程實作 + Python agent + Next.js UI」的專案。核心執行邏輯在 asset_lifecycle_management_agent，教學流程在根目錄 notebooks，前端介面在 NAT_Toolkit-UI。

**根目錄檔案**

- 0_ALM_Database_Setup.ipynb: 教你下載 NASA C-MAPSS、建立 SQLite 資料庫。
- 1_ALM_Application_Setup.ipynb: 教你安裝與啟動 ALM agent。
- 2_ALM_Config_Understanding.ipynb: 解釋 `config-reasoning.yml` 的模型、工具、workflow 設定。
- 3_ALM_Observability.ipynb: 示範 tracing / observability。
- 4_ALM_Evaluation.ipynb: 評估 agent 回答品質。
- 5_ALM_AnomalyDetection.ipynb: 異常偵測實作與 MOMENT/NV-Tesseract 使用。
- Optional_ALM_Codegen.ipynb: 選做的程式碼生成練習。
- nat_client.py: notebook 用的控制客戶端，負責 `start/stop/restart/query/show/logs/reset` NAT 後端。
- setup_database.py: 下載 NASA 資料集、解析文字檔、寫入 SQLite。
- configs/config-reasoning.yml: 主設定檔，定義 LLM、embedder、各工具與 reasoning workflow。
- eval_set_lab.json、eval_set_lab-Copy1.json: 評測題集。
- models/scaler_model.pkl: RUL 模型前處理 scaler。
- models/xgb_model_fd001.pkl: XGBoost RUL 預測模型。
- images/*: 課程投影片/說明圖片。
- .asset_lifecycle_management_agent_backup/*: 練習重置用的備份版本。
- .ipynb_checkpoints/*: Jupyter 自動儲存檢查點。

**Python 套件 `asset_lifecycle_management_agent`**

- README.md: 專案用途、架構、安裝流程。
- pyproject.toml: Python 套件定義、依賴、 NAT 元件入口。
- vanna_training_data.yaml: 提供給 Vanna 的 SQL/schema few-shot 訓練資料。
- src/asset_lifecycle_management_agent/__init__.py: 套件版本。
- src/asset_lifecycle_management_agent/register.py: NAT 啟動時自動註冊所有工具與 evaluator 的入口。

**`retrievers/`**

- `__init__.py`: 套件標記。
- `generate_sql_query_and_retrieve_tool.py`: 將自然語言需求轉成 SQL，查資料庫並輸出結果。
- `vanna_manager.py`: 管理 Vanna、向量庫與 SQL 生成流程。
- `vanna_util.py`: Vanna 相關輔助函式。

**`predictors/`**

- `__init__.py`: 套件標記。
- `predict_rul_tool.py`: 用 scaler + XGBoost 做剩餘壽命預測。
- `nv_tesseract_anomaly_detection_tool.py`: 透過 NVIDIA NIM / NV-Tesseract 做異常偵測。
- `moment_anomaly_detection_tool.py`: 用 MOMENT 模型做異常偵測的替代實作。
- `moment_predict_rul_tool.py`: 用 MOMENT 做 RUL 預測的替代實作。

**`plotting/`**

- `__init__.py`: 套件標記。
- `plot_utils.py`: 畫圖共用函式。
- `plot_line_chart_tool.py`: 畫單條折線圖。
- `plot_comparison_tool.py`: 畫多序列比較圖。
- `plot_distribution_tool.py`: 畫分布/直方圖。
- `plot_anomaly_tool.py`: 將異常偵測結果視覺化。
- `code_generation_assistant.py`: 選做題用，讓 agent 生成輔助程式碼。

**`evaluators/`**

- `__init__.py`: 套件標記。
- `llm_judge_evaluator.py`: 純文字評審 evaluator。
- `llm_judge_evaluator_register.py`: 註冊文字 evaluator。
- `multimodal_llm_judge_evaluator.py`: 支援圖片/多模態輸入的 evaluator。
- `multimodal_llm_judge_evaluator_register.py`: 註冊多模態 evaluator。


**練習答案與評量**

- Exercise_Solutions/Exercise_1_Temperature/config-reasoning.yml: Exercise 1 的完成版設定。
- Exercise_Solutions/Exercise_2_Plotting/plot_line_chart_tool.py、plot_utils.py: Exercise 2 的完成版畫圖工具。
- Exercise_Solutions/Exercise_2_Plotting/config-reasoning.yml: Exercise 2 完成版設定。
- Exercise_Solutions/Exercise_3_Action_Recommendation/recommenders/action_recommendation_tool.py: 根據資料結果給維護建議。
- `Exercise_Solutions/Exercise_3_Action_Recommendation/register.py`: 註冊 action recommendation 工具。
- `Exercise_Solutions/Exercise_3_Action_Recommendation/recommenders/__init__.py`: 套件標記。
- `Exercise_Solutions/Exercise_3_Action_Recommendation/config-reasoning.yml`: Exercise 3 完成版設定。
- Exercise_Solutions/Exercise_4_Evaluation/eval_set_custom.json: 自訂評測集。
- Exercise_Solutions/Exercise_Optional_CodeGen/config-reasoning.yml: 選做 codegen 完成版設定。
- Assessment/ALM_assessment.ipynb: 課程評量 notebook。
- Assessment/plot_line_chart_with_ma_tool.py: 評量題要完成的帶移動平均畫圖工具。
- Assessment/backup/plot_line_chart_with_ma_tool-TEMPLATE.py: 模板備份。
- `Assessment/images/*`: 評量畫面截圖。