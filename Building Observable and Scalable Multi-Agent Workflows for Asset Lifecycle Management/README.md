```yaml
embedders:
  # Text embedding model for vector database operations
  vanna_embedder:
    _type: nim
    model_name: "nvidia/llama-nemotron-embed-1b-v2"
    base_url: ${NVIDIA_BASE_URL}
```

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