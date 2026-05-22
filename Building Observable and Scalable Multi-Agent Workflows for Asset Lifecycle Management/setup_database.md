`setup_database.py` 的目的：

**把 NASA C-MAPSS Turbofan Engine Dataset 的 `.txt` 檔轉成 SQLite database**，方便後續用 SQL 查詢、做 Remaining Useful Life（RUL）預測、異常偵測或 Asset Lifecycle Management agent 使用。

---

## 1. 整體流程

程式執行後大致做這些事：

```text
檢查 NASA dataset 是否存在
        ↓
如果不存在，從 NASA URL 下載 CMAPSSData.zip
        ↓
解壓縮到 nasa_dataset_raw/
        ↓
讀取 train_FD001 ~ train_FD004
        ↓
讀取 test_FD001 ~ test_FD004
        ↓
讀取 RUL_FD001 ~ RUL_FD004
        ↓
建立 SQLite database
        ↓
建立 metadata tables、整合 tables、indexes、views
        ↓
執行 validation queries 確認資料是否成功建立
```

最後會產生：

```text
database/nasa_turbo.db
```

---

## 2. 主要 class：`NASADatasetProcessor`

```python
class NASADatasetProcessor:
```

這是整支程式的核心 class，負責：

1. 檢查或下載 NASA dataset
2. 讀取 `.txt` 資料
3. 轉成 pandas DataFrame
4. 寫入 SQLite database
5. 建立 metadata、index、view
6. 驗證 database 是否正確建立

---

## 3. `__init__()`：初始化設定

```python
def __init__(self, data_dir="nasa_dataset_raw", db_path="database/nasa_turbo.db"):
```

這裡設定兩個重要路徑：

```python
self.data_dir = Path(data_dir)
self.db_path = Path(db_path)
```

意思是：

| 變數         | 用途                      |
| ---------- | ----------------------- |
| `data_dir` | 存放 NASA 原始 `.txt` 檔的資料夾 |
| `db_path`  | SQLite database 輸出位置    |

預設值是：

```text
nasa_dataset_raw/
database/nasa_turbo.db
```

另外這段：

```python
self.db_path.parent.mkdir(exist_ok=True)
```

會確保 `database/` 資料夾存在。如果不存在，就自動建立。

---

## 4. `self.columns`：定義資料欄位名稱

NASA C-MAPSS 原始 `.txt` 沒有 header，所以程式自己指定 column 名稱：

```python
self.columns = [
    'unit_number', 'time_in_cycles',
    'operational_setting_1', ...
    'sensor_measurement_1', ... 'sensor_measurement_21'
]
```

每一筆資料代表某個引擎在某個 cycle 的狀態。

主要欄位意思：

| column                    | 意思         |
| ------------------------- | ---------- |
| `unit_number`             | 引擎 ID      |
| `time_in_cycles`          | 目前運轉 cycle |
| `operational_setting_1~3` | 操作條件       |
| `sensor_measurement_1~21` | 21 個感測器讀值  |

例如：

```text
unit_number = 1
time_in_cycles = 50
sensor_measurement_2 = 某個溫度感測器讀值
```

意思是：
**第 1 台引擎，在第 50 個 cycle 的感測器狀態。**

---

## 5. `sensor_descriptions`：感測器說明表

```python
self.sensor_descriptions = {
    'sensor_measurement_1': 'Total temperature at fan inlet (°R)',
    ...
}
```

這是 metadata，用來說明每個 sensor column 代表什麼。

之後會被寫入 SQLite 的：

```text
sensor_metadata
```

table。

例如：

| sensor_name             | description                         |
| ----------------------- | ----------------------------------- |
| `sensor_measurement_1`  | Total temperature at fan inlet      |
| `sensor_measurement_8`  | Physical fan speed                  |
| `sensor_measurement_21` | Low-pressure turbines Cool air flow |

---

## 6. `download_nasa_dataset()`：下載資料集

```python
def download_nasa_dataset(self) -> bool:
```

這個 function 做三件事：

### Step 1：檢查檔案是否已存在

它要求這 12 個檔案都要存在：

```text
train_FD001.txt ~ train_FD004.txt
test_FD001.txt ~ test_FD004.txt
RUL_FD001.txt ~ RUL_FD004.txt
```

如果都存在，就不下載：

```python
return True
```

### Step 2：如果缺檔，就下載 ZIP

```python
nasa_dataset_url = "https://data.nasa.gov/docs/legacy/CMAPSSData.zip"
```

它用：

```python
requests.get(..., stream=True, timeout=600)
```

下載資料。

這裡 `stream=True` 代表不是一次把整個 ZIP 載入記憶體，而是分 chunk 下載，對大檔案比較安全。

### Step 3：解壓縮

```python
with zipfile.ZipFile(tmp_zip_path, 'r') as zip_ref:
    zip_ref.extractall(self.data_dir)
```

解壓縮到：

```text
nasa_dataset_raw/
```

---

## 7. `read_data_file()`：讀取 train/test `.txt`

```python
df = pd.read_csv(file_path, sep='\s+', header=None, names=self.columns)
```

這行很重要。

NASA 原始資料是空白分隔，例如：

```text
1 1 -0.0007 -0.0004 100.0 518.67 ...
```

所以用：

```python
sep='\s+'
```

代表用「一個或多個空白」切開資料。

因為原始檔沒有 column header，所以：

```python
header=None
names=self.columns
```

表示由程式自己補上 column 名稱。

---

## 8. `process_training_data()`：處理訓練資料

```python
training_files = [
    'train_FD001.txt', 'train_FD002.txt', 'train_FD003.txt', 'train_FD004.txt'
]
```

它會逐一讀取四個 training file。

重點是這一行：

```python
df['RUL'] = df.groupby('unit_number')['time_in_cycles'].transform('max') - df['time_in_cycles']
```

這是在計算 training data 的 RUL。

### 為什麼可以這樣算？

training data 是 **run-to-failure trajectory**，也就是每台引擎都一路記錄到壞掉為止。

假設某台引擎最多跑到 cycle 200：

| time_in_cycles | RUL |
| -------------: | --: |
|              1 | 199 |
|             50 | 150 |
|            100 | 100 |
|            199 |   1 |
|            200 |   0 |

所以：

```text
RUL = 該引擎最大 cycle - 目前 cycle
```

例如：

```text
max cycle = 200
current cycle = 50
RUL = 200 - 50 = 150
```

接著它會寫成 SQLite table：

```python
table_name = file_name.replace('.txt', '')
df.to_sql(table_name, conn, if_exists='replace', index=False)
```

所以會建立：

```text
train_FD001
train_FD002
train_FD003
train_FD004
```

---

## 9. `process_test_data()`：處理測試資料

```python
test_files = [
    'test_FD001.txt', 'test_FD002.txt', 'test_FD003.txt', 'test_FD004.txt'
]
```

test data 也會讀進 SQLite，但它不會計算每個 cycle 的 RUL。

原因是：
**test data 不是完整跑到 failure 的 trajectory**，它只給你某個時間點以前的感測器資料。

所以 test table 只會有 sensor data：

```text
test_FD001
test_FD002
test_FD003
test_FD004
```

真正的 test RUL 會放在 `RUL_FDxxx.txt` 裡。

---

## 10. `process_rul_data()`：處理測試資料的 RUL label

```python
rul_values = pd.read_csv(file_path, header=None, names=['RUL'])
rul_values['unit_number'] = range(1, len(rul_values) + 1)
```

RUL file 每個 row 是一台 test engine 最後一個觀測 cycle 後的真實剩餘壽命。

例如 `RUL_FD001.txt` 可能長這樣：

```text
112
98
69
...
```

它代表：

| unit_number | RUL |
| ----------: | --: |
|           1 | 112 |
|           2 |  98 |
|           3 |  69 |

程式用：

```python
range(1, len(rul_values) + 1)
```

幫每個 RUL 加上對應的 `unit_number`。

最後建立：

```text
RUL_FD001
RUL_FD002
RUL_FD003
RUL_FD004
```

---

## 11. `create_metadata_tables()`：建立說明資料表

這個 function 建立兩個 metadata tables。

### `sensor_metadata`

記錄每個 sensor column 的物理意義。

### `dataset_metadata`

```python
dataset_metadata = pd.DataFrame([
    {'dataset': 'FD001', 'description': 'Sea level conditions', 'fault_modes': 1},
    {'dataset': 'FD002', 'description': 'Sea level conditions', 'fault_modes': 6},
    {'dataset': 'FD003', 'description': 'High altitude conditions', 'fault_modes': 1},
    {'dataset': 'FD004', 'description': 'High altitude conditions', 'fault_modes': 6}
])
```

這裡把 FD001~FD004 的條件寫進 database。

| dataset | description              | fault_modes |
| ------- | ------------------------ | ----------: |
| FD001   | Sea level conditions     |           1 |
| FD002   | Sea level conditions     |           6 |
| FD003   | High altitude conditions |           1 |
| FD004   | High altitude conditions |           6 |

用途是讓 agent 或 SQL query 可以知道每個子資料集的背景。

---

## 12. `create_consolidated_tables()`：建立整合版 tables

前面是分開建：

```text
train_FD001
train_FD002
train_FD003
train_FD004
```

這個 function 會把它們合併成：

```text
training_data
```

核心程式：

```python
parts = [
    f"SELECT *, '{d}' AS dataset FROM train_{d}" for d in datasets
]
```

這會產生：

```sql
SELECT *, 'FD001' AS dataset FROM train_FD001
UNION ALL
SELECT *, 'FD002' AS dataset FROM train_FD002
UNION ALL
...
```

所以整合後的 `training_data` 會多一個 column：

```text
dataset
```

用來標記這筆 row 來自 FD001、FD002、FD003 或 FD004。

同理，它也會建立：

```text
test_data
rul_data
```

---

## 13. `create_indexes()`：建立索引，加速查詢

```python
CREATE INDEX IF NOT EXISTS idx_train_FD001_unit ON train_FD001(unit_number)
CREATE INDEX IF NOT EXISTS idx_train_FD001_cycle ON train_FD001(time_in_cycles)
```

這些 index 是為了讓常見查詢更快，例如：

```sql
SELECT * FROM train_FD001 WHERE unit_number = 10;
```

或：

```sql
SELECT * FROM train_FD001 WHERE time_in_cycles > 100;
```

它對這些 column 建 index：

| table         | indexed column                  |
| ------------- | ------------------------------- |
| `train_FDxxx` | `unit_number`, `time_in_cycles` |
| `test_FDxxx`  | `unit_number`, `time_in_cycles` |
| `RUL_FDxxx`   | `unit_number`                   |

---

## 14. `create_views()`：建立常用查詢 view

這裡建立兩個 SQLite view。

---

### View 1：`latest_sensor_readings`

```sql
CREATE VIEW IF NOT EXISTS latest_sensor_readings AS
SELECT t1.*
FROM training_data t1
INNER JOIN (
    SELECT unit_number, dataset, MAX(time_in_cycles) as max_cycle
    FROM training_data
    GROUP BY unit_number, dataset
) t2 ON ...
```

這個 view 會找出每台 engine 在 training data 中的最後一筆感測器資料。

也就是：

```text
每台引擎壞掉前最後一個 cycle 的狀態
```

用途：

```sql
SELECT * FROM latest_sensor_readings;
```

可以快速看每台 engine 最終狀態。

---

### View 2：`engine_health_summary`

```sql
CREATE VIEW IF NOT EXISTS engine_health_summary AS
SELECT 
    unit_number,
    dataset,
    MAX(time_in_cycles) as total_cycles,
    MIN(RUL) as final_rul,
    AVG(sensor_measurement_1) as avg_fan_inlet_temp,
    AVG(sensor_measurement_11) as avg_hpc_outlet_pressure,
    AVG(sensor_measurement_21) as avg_lpt_cool_air_flow
FROM training_data
GROUP BY unit_number, dataset
```

這個 view 對每台 engine 做摘要。

例如：

| column                    | 意思                            |
| ------------------------- | ----------------------------- |
| `total_cycles`            | 這台 engine 總共跑了幾個 cycles       |
| `final_rul`               | 最後 RUL，理論上 training data 會是 0 |
| `avg_fan_inlet_temp`      | sensor 1 平均值                  |
| `avg_hpc_outlet_pressure` | sensor 11 平均值                 |
| `avg_lpt_cool_air_flow`   | sensor 21 平均值                 |

---

## 15. `validate_database()`：驗證資料庫

它會執行幾個 SQL query：

```python
validation_queries = [
    ("Training data count", "SELECT COUNT(*) FROM training_data"),
    ("Test data count", "SELECT COUNT(*) FROM test_data"),
    ("RUL data count", "SELECT COUNT(*) FROM rul_data"),
    ("Unique engines in training", "SELECT COUNT(DISTINCT unit_number) FROM training_data"),
    ("Datasets available", "SELECT DISTINCT dataset FROM training_data"),
]
```

目的不是做模型驗證，而是確認 database 有沒有成功建立。

例如它會印出：

```text
Training data count: xxxx
Test data count: xxxx
RUL data count: xxxx
Datasets available: ['FD001', 'FD002', 'FD003', 'FD004']
```

---

## 16. `process_dataset()`：主流程控制

這是整個 class 最重要的 orchestration function。

```python
def process_dataset(self):
```

執行順序是：

```python
self.download_nasa_dataset()
self.process_training_data(conn)
self.process_test_data(conn)
self.process_rul_data(conn)
self.create_metadata_tables(conn)
self.create_consolidated_tables(conn)
self.create_indexes(conn)
self.create_views(conn)
self.validate_database(conn)
```

也就是：

```text
下載 / 檢查資料
→ 建立 per-dataset tables
→ 建立 metadata
→ 建立 consolidated tables
→ 建立 indexes
→ 建立 views
→ 驗證 database
```

---

## 17. `main()`：命令列入口

```python
parser.add_argument("--data-dir", default="nasa_dataset_raw")
parser.add_argument("--db-path", default="database/nasa_turbo.db")
```

代表你可以這樣執行：

```bash
python setup_database.py
```

也可以指定路徑：

```bash
python setup_database.py \
    --data-dir ./CMAPSSData \
    --db-path ./database/nasa_turbo.db
```

最後：

```python
if __name__ == "__main__":
    exit(main())
```

意思是：
當你直接執行這個 `.py` 檔時，就呼叫 `main()`。

---

## 18. 這支程式建立的 database 結構

執行成功後，SQLite database 會包含：

### 原始分開 tables

```text
train_FD001
train_FD002
train_FD003
train_FD004

test_FD001
test_FD002
test_FD003
test_FD004

RUL_FD001
RUL_FD002
RUL_FD003
RUL_FD004
```

### 整合 tables

```text
training_data
test_data
rul_data
```

### Metadata tables

```text
sensor_metadata
dataset_metadata
```

### Views

```text
latest_sensor_readings
engine_health_summary
```

---

## 19. 一個重要觀念：training RUL vs test RUL

### Training data

Training data 是完整壽命資料，所以可以直接算：

```python
RUL = max_cycle - current_cycle
```

### Test data

Test data 不知道後面還能跑多久，所以不能從 test data 自己算 RUL。
因此 NASA 另外提供：

```text
RUL_FD001.txt
RUL_FD002.txt
RUL_FD003.txt
RUL_FD004.txt
```

作為 ground truth label。

---

## 20. 簡短總結

這支程式是 NASA C-MAPSS dataset 的 **ETL pipeline**：

```text
Extract：下載 / 讀取 NASA txt dataset
Transform：加上 column name、計算 training RUL、整理 metadata
Load：寫入 SQLite database
```

最核心的價值是：
把原本分散的 `.txt` 檔整理成可以 SQL 查詢的 database，方便後續做：

```text
RUL prediction
predictive maintenance
engine degradation analysis
sensor trend analysis
asset lifecycle agent
```