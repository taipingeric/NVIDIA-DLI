# 課程架構（Course Structure）

本課程共分為以下 **4 個 Notebook**。雖然每個 Notebook 都可以獨立執行，但**建議依照順序逐一學習**。

---

## Chapter 1：簡介（Introduction）

本 Notebook 將介紹**聯邦學習（Federated Learning, FL）**與 **NVIDIA FLARE** 的基本概念，提供整體性的入門概覽。

---

## Chapter 2：開發聯邦學習應用程式（Develop a Federated Application）

本 Notebook 著重於如何利用 **NVIDIA FLARE** 提供的 API 與工具開發聯邦學習應用程式。

內容包括：

* 介紹 NVIDIA FLARE 的高階架構（High-level Architecture）
* 說明開發聯邦學習 **Server、Client 與 Job** 所需的基本 API
* 透過實例示範如何將傳統的**集中式（Centralized）**程式碼，輕鬆改寫為**聯邦學習（Federated）**程式碼
* 展示如何利用 **FLARE Simulator** 在本機的模擬環境中快速執行聯邦學習應用程式

---
```
examples/hello-fedavg-numpy
├── README.md
├── fedavg_script_runner_hello-numpy.py
├── hello-fedavg-numpy_flare_api.ipynb
├── hello-fedavg-numpy_getting_started.ipynb
├── requirements.txt
└── src
    └── hello-numpy_fl.py
```

* src/hello-numpy_fl.py: 實作 Executor，負責定義每個站點（Site）的本地運算流程（Local Computation Routine）。我們之所以先介紹 Client 端，是因為資料科學家通常都是從既有的集中式（Centralized）程式開始開發，再將其改造成聯邦學習版本。我們希望藉由這個範例，展示使用 NVIDIA FLARE 的 Client API，如何能夠簡單且順暢地將傳統程式轉換為聯邦學習程式

* fedavg_script_runner_hello-numpy.py
** 建立 Controller，用來定義伺服器端的模型聚合（Aggregation）工作流程
** 如何把前面建立的 Server 端 Controller 與 Client 端 Executor 連接起來，並包裝成一個 Federated Job（聯邦工作），使其能夠由 NVIDIA FLARE 執行整個聯邦學習流程。

* Client-Side 用戶端實作: src/hello-numpy_fl.py

在聯邦學習中，Client 通常會實作 Executor，負責處理每個用戶端站點（Client Site）本地執行的運算流程或訓練邏輯（Computation Routine / Training Logic）。在傳統的** 集中式（Centralized）**運算模式中，這裡就是開發者撰寫核心運算或模型訓練程式碼的位置

* Server-Side 伺服器端的工作流程。

如同前面介紹 NVIDIA FLARE 架構時所說，**伺服器端的主要工作是建立 Controller**。Controller 用來定義伺服器端的協調（orchestration）與聚合（aggregation）流程，以及與各個 Client 互動的邏輯。

相較於傳統的集中式運算，**伺服器端程式碼是一個全新的組成部分**，因此需要由開發者自行實作。不過，NVIDIA FLARE 提供了一套簡單且容易擴充的方式來完成這件事。

以下是檔案：

```text
examples/hello-fedavg-numpy/fedavg_script_runner_hello-numpy.py
```

中與伺服器端實作相關的程式碼：

```python
n_clients = 2
num_rounds = 3

persistor_id = job.to_server(NPModelPersistor(), "persistor")

# Define the controller workflow and send to server
controller = FedAvg(
    num_clients=n_clients,
    num_rounds=num_rounds,
    persistor_id=persistor_id,
)
```

可以看到，伺服器端程式其實只是建立了一個 NVIDIA FLARE 提供的 **`FedAvg`** 類別。

原因很簡單，在這個範例中，伺服器端的工作流程非常直接：**接收各個 Client 回傳的結果，並計算它們的平均值**。

這正是經典的 **Federated Averaging（FedAvg，聯邦平均）** 演算法，因此可以直接使用 NVIDIA FLARE 已經提供的 `FedAvg Controller`。

---

`FedAvg` Controller 建立時需要三個參數：

* **`num_clients`**

  表示每一輪訓練要選擇多少個 Client 參與。

  這個數值不一定等於實際參與系統的 Client 總數。

  * 不能大於 Client 總數。
  * 可以小於 Client 總數。

  如果小於總數，則每一輪會**隨機抽取（random subset）** `num_clients` 個 Client 參與訓練。

  值得注意的是，在 FLARE 中，Client 的抽樣策略（sampling strategy）是可以完全客製化的，後續會再介紹。

---

* **`num_rounds`**

  表示總共要執行多少輪聚合（aggregation）。

  它同時也決定了 Client 端程式中：

  ```python
  while flare.is_running():
  ```

  這段迴圈內的運算會執行多少次。

---

* **`persistor_id`**

  表示一個 **Persistor** 物件的 ID。

---

在 NVIDIA FLARE 中，**Persistor** 的職責是負責資料的**載入（loading）**與**序列化（serialization）**。在目前的情境下，它主要管理的是**伺服器端的全域模型（global model）**。

本範例中的全域模型只是一個 `numpy` 陣列，因此可以直接使用 NVIDIA FLARE 提供的 **`NPModelPersistor`**。

它主要實作了以下功能：

* **預設陣列初始化（Default array initialization）**

  通常用於聯邦學習的第一輪（initial round）。

  在第一輪時，Server 尚未收到任何 Client 的模型，因此需要先建立一個預設模型，再傳送給各個 Client。

---

* **`save_model()`**

  將模型序列化並儲存到檔案系統中，格式為 `numpy` 的 `.npy` 檔案。

---

* **`load_model()`**

  從檔案系統讀取 `.npy` 檔案，並載入成 `numpy` 陣列。

---

建立好的 Persistor，其 ID 會傳入 `FedAvg` Controller。

因此，當 Controller 需要：

* 初始化全域模型、
* 將模型儲存到磁碟、
* 從磁碟載入模型，

便會自動呼叫 Persistor 中對應的方法。

---

除了 `NPModelPersistor` 之外，FLARE 還提供許多現成的模型 Persistor，例如：

* **`PTFileModelPersistor`**

  用於 **PyTorch** 模型。

* **`TFModelPersistor`**

  用於 **TensorFlow** 模型。

此外，如果現有的 Persistor 不符合需求，也可以透過繼承（subclass）**`ModelPersistor`** 類別，自行實作自己的模型 Persistor。

另一方面，也**不一定非得使用 Persistor** 來完成模型的儲存與載入。FLARE 的基礎 `Controller` 類別具有相當高的彈性，只要覆寫（override）相應的方法，就可以自行實作模型保存與讀取的邏輯。

因此，在這個教學範例中，只需建立現成的 **`FedAvg` Controller**，就能快速完成伺服器端的 **Federated Averaging（聯邦平均）** 工作流程設定。

將所有元件組合成一個 Federated Job

現在我們已經完成了 Client 與 Server 端的實作，剩下要做的事情，就是**將所有元件整合成一個 Federated Job（聯邦工作）**，讓它能夠由 NVIDIA FLARE 的 Runtime 執行。

這可以透過 **FedJob API** 很容易地完成。

下面來看看完整的檔案：

```text
examples/hello-fedavg-numpy/fedavg_script_runner_hello-numpy.py
```

（程式碼順序經過調整，以方便說明。）

---

首先，程式匯入 `FedJob` 類別，並建立一個 `FedJob` 物件：

```python
from nvflare import FedJob
...
job = FedJob(name="hello-fedavg-numpy")
```

`FedJob` 的目的，是用來描述整個聯邦學習工作流程（Federated Workflow）以及 Server 與 Client 之間的互動，並提供 Python 風格（Pythonic）的 API，讓使用者可以方便地建立與設定聯邦工作。

---

接下來幾行程式，是先前介紹過的 Server 端設定：

```python
n_clients = 2
num_rounds = 3

persistor_id = job.to_server(NPModelPersistor(), "persistor")

controller = FedAvg(
    num_clients=n_clients,
    num_rounds=num_rounds,
    persistor_id=persistor_id,
)
job.to(controller, "server")
```

這裡設定：

* 每一輪共有 **2 個 Client** 參與訓練。
* 整個聯邦學習共執行 **3 個 round**。

---

這段程式也展示了 `FedJob.to()` API 的用途。

`FedJob.to()` 的功能是：

> **將某個物件送到指定的目標（target）執行。**

target 可以是：

* Server
* Client

在 FLARE 中，每個建立好的元件，都必須送到它所屬的執行端，例如：

* Controller、Persistor → Server
* Executor → Client

因此：

```python
job.to_server(NPModelPersistor(), "persistor")
```

等價於：

```python
job.to(NPModelPersistor(), "server")
```

只是額外將它指定一個 ID：

```
persistor
```

所以可以理解為：

* `job.to_server(NPModelPersistor(), "persistor")`

  將一個 `NPModelPersistor` 實例送到 Server，並命名為 `"persistor"`。

---

而：

```python
job.to(controller, "server")
```

表示：

將 `FedAvg Controller` 部署到 Server。

它也等價於：

```python
job.to_server(controller)
```

---

接著：

```python
job.to(
    IntimeModelSelector(
        key_metric="accuracy"
    ),
    "server"
)
```

又將另一個元件：`IntimeModelSelector`送到 Server。本課程不深入介紹這個元件，但只需要知道：它的功能是在聯邦學習過程中，根據某個評估指標，自動挑選並保存最佳的 Global Model。這裡使用的評估指標是`accuracy`。之所以能做到，是因為每個 Client 回傳的是一個 `FLModel` 物件，其中包含：`metrics`屬性，而 `IntimeModelSelector` 就會根據這些 metrics 判斷哪一個模型最好。

在目前這個範例中，由於 evaluation 只是示意性的 mock evaluation，因此用 accuracy 選最佳模型沒有太大的實際意義。不過，在後續較真實的案例中，就能看到它的用途。

---

接下來介紹 Client 端的部分：

```python
train_script = "src/hello-numpy_fl.py"

for i in range(n_clients):
    executor = ScriptRunner(
        script=train_script,
        script_args="",
        framework=FrameworkType.NUMPY,
    )

    job.to(executor, f"site-{i+1}")
```

可以把它理解成：

> 利用 `ScriptRunner`，把原本的 Client Python 程式轉換成一個 Executor。

原本寫好的：

```text
examples/hello-fedavg-numpy/src/hello-numpy_fl.py
```

透過：

```python
ScriptRunner(...)
```

包裝成 FLARE 可以執行的 Executor。

然後：

```python
job.to(...)
```

再把它送到每一個 Client：

```
site-1
site-2
...
```

實際上，`ScriptRunner` 內部的實作比這裡描述得更複雜，但概念上可以這樣理解。

最大的好處是：

開發者**不需要自己手動撰寫 Executor 類別**。

而是：

> 先從原本的集中式訓練程式開始 → 加入 FLARE Client API → 再透過 `ScriptRunner` 自動轉換成 Executor。

因此，大幅降低了將既有程式改造成聯邦學習程式的難度。

---

至此，一個 Federated Job 的定義與設定就完成了。

接著有兩種使用方式。

第一種，直接利用 **FL Simulator** 執行：

```python
job.simulator_run(
    "/tmp/nvflare/jobs/workdir",
    gpu="0",
)
```

這會在模擬環境中立即執行聯邦學習。

---

第二種，先將 Job 匯出：

```python
job.export_job(
    "/tmp/nvflare/jobs/job_config"
)
```

之後再利用不同的 Runtime Backend 執行。

---

本教學接下來採用第二種方式。

因此，需要：

1. 將`job.export_job(...)`取消註解（uncomment）。
2. 將`job.simulator_run(...)`註解掉（comment）。

修改完成後，再執行目前的程式即可。

| 比較項目             | FedAvg                   | Cyclic                 |
| ---------------- | ------------------------ | ---------------------- |
| 模型更新方式           | Server 收集多個 Client 結果後平均 | 模型依序在 Client 間傳遞       |
| 每一輪參與者           | 多個 Client 同時參與           | 一次通常只有一個 Client        |
| 是否需要 Aggregation | ✔ 需要                     | ✘ 不需要                  |
| Server 工作        | 收集並平均模型                  | 只負責控制模型流向              |
| 通訊模式             | Parallel（平行）             | Sequential（序列）         |
| 經典演算法            | Federated Averaging      | Cyclic Weight Transfer |


## Chapter 3：建立、執行與監控聯邦學習專案（Provision, Run and Monitor a Federated Project）

本 Notebook 著重於如何正確建立聯邦學習專案，並在沙盒（Sandbox）環境中執行。

內容包括：

* 深入介紹 NVIDIA FLARE 中的**專案建置（Project Provisioning）**概念
* 示範如何使用 **Proof-of-Concept（PoC）模式**，在本機環境測試、執行與監控已建置完成的聯邦學習專案
* 模擬真實世界部署情境
* 透過範例說明專案建置、執行與監控的完整流程

---

在第 2 章中，我們學習了如何利用 **NVIDIA FLARE** 的 API 開發一個聯邦學習應用程式，並使用 **FL Simulator** 在模擬環境中執行它。

在那個範例中，我們定義了：

* 1 個 Server
* 2 個 Client

作為聯邦學習專案（FL Project）的參與者（participants）。

這些參與者的建立與管理，都由 **FL Simulator** 自動完成，因此我們不需要自行設定。雖然這種方式非常適合原型開發（prototyping），但在真實世界的聯邦學習部署中，通常需要更細緻的控制。

事實上，在實際的聯邦學習部署中，不同參與者都有各自明確的角色。

除了 **Server** 與 **Client** 之外，一個 FL 專案通常還會配置：

* **Administrator（管理員）**

  負責管理與監控整個專案的執行狀態。

此外，還需要建立完善的：

* 使用者身分驗證（Authentication）
* 使用者授權（Authorization）

機制，以確保所有參與者能夠在安全的環境下建立彼此信任的通訊管道（mutually trusted communication channels）。

另一方面，由於聯邦學習本身具有**分散式（distributed）**的特性，各個參與者可能位於不同地點，因此需要透過安全的網路連線加入 FL 系統，而且這些連線甚至可能是動態建立的。

除此之外，不同參與者在：

* 隱私保護（Privacy）
* 運算資源（Compute Resources）

方面也可能有不同需求，因此系統還需要提供額外的安全性與資源管理功能。

為了能夠妥善處理上述問題，NVIDIA FLARE 提出了 **Provisioning（專案建置）** 的概念。

Provisioning 指的是：

> **建立聯邦學習專案及其所有參與者，使其能夠在真實環境中以可擴充（scalable）、可延伸（extensible）且安全（secure）的方式部署的整個流程。**

在 NVIDIA FLARE 中，Provisioning 的主要用途包括：

* 建立 Server、Client 與 Administrator 的身分識別（Identity）
* 為每個參與者產生啟動套件（Startup Kit）
* 建立參與者之間的安全通訊機制（Secure Communication Channels）

Provisioning 可以透過兩種方式完成：

1. 使用命令列工具（CLI）

```text
nvflare provision
```

2. 使用 Web 介面的 **FLARE Dashboard**

本章將僅介紹 **CLI 工具**的使用方式，而 FLARE Dashboard 則會在下一章簡要介紹。

---

Provisioning 的流程通常包含以下三個步驟：

1. **使用設定檔（Configuration File）定義整個專案**

   包括 Server、Client、Administrator 等參與者及相關設定。

2. **利用 FLARE Provisioning 工具，根據設定檔產生 Startup Kit**

   Startup Kit 是每個參與者啟動系統所需的設定與憑證。

3. **將各自的 Startup Kit 分發給對應的參與者**

   參與者取得自己的 Startup Kit 後，即可加入聯邦學習系統並建立安全連線。

接下來，我們將透過一個實際範例，一步一步介紹上述流程。

範例：建立（Provision）一個聯邦學習專案

**1. 設定專案（Configure the project）**

Provisioning 流程的輸入是一個**專案設定檔（Project Configuration File）**，通常是一個 `yaml` 檔案。

這個設定檔用來定義整個聯邦學習專案，通常包含以下內容：

* 專案中繼資訊（Project Meta-data）
* 專案參與者（Participants）
* 用於建立專案工作空間的 Builder（Builders）

---

範例設定檔位於：

```text
../files/project.yml
```

此檔案是直接執行：

```bash
nvflare provision
```

且**不帶任何參數**時所產生的。

此外，官方也提供更完整的 `project.yml` 範本，可作為建立專案的參考。

---

**注意（NOTE）**

當執行 `nvflare provision` 而未提供任何參數時，系統會詢問是否要在產生的設定檔中加入 **High Availability（高可用性）** 功能。

本課程不討論 High Availability，因此此範例是在**未啟用 High Availability** 的情況下產生。

---

下面一起看看 `project.yml` 的主要內容（為方便說明，已省略部分細節與註解）：

```yaml
api_version: 3
name: example_project
description: NVIDIA FLARE sample project yaml file

participants:
  - name: server1
    type: server
    org: nvidia
    fed_learn_port: 8002
    admin_port: 8003

  - name: site-1
    type: client
    org: nvidia

  - name: site-2
    type: client
    org: nvidia

  - name: admin@nvidia.com
    type: admin
    org: nvidia
    role: project_admin

builders:
  - path: nvflare.lighter.impl.workspace.WorkspaceBuilder
      ...
  - path: nvflare.lighter.impl.static_file.StaticFileBuilder
      ...
  - path: nvflare.lighter.impl.cert.CertBuilder
      ...
  - path: nvflare.lighter.impl.signature.SignatureBuilder
```

整個設定檔可以分成三個部分：

---

**Project Meta-data（專案中繼資訊）**

這部分描述整個聯邦學習專案的基本資訊，包括：

* `api_version`

  指定使用的 Provisioning API 版本。

  在目前 NVIDIA FLARE 的版本中，`api_version` 設定為：

  ```yaml
  api_version: 3
  ```

* `name`

  專案名稱。

  本例中為：

  ```yaml
  example_project
  ```

* `description`

  專案的簡短描述。

---

**Participants（參與者）**

這是整個設定檔中最重要的部分之一，用來定義所有參與聯邦學習專案的角色（participants）。

FLARE 支援多種類型的參與者，最常見的包括：

* `server`
* `client`
* `admin`

其中，有些參與者屬於 **Site**（站點），代表真正執行 FLARE 應用程式的運算系統，例如：

* Server
* Client

另外一些參與者則屬於 **User**（使用者），也就是實際的人員，他們擁有不同權限，可以查詢、監控或管理專案，例如：

* Admin

除此之外，FLARE 還支援其他角色，例如：

* `overseer`

  用於 High Availability 模式。

甚至也可以自行定義新的 Participant 類型，但本課程不介紹這些進階功能。

---

在目前的 `project.yml` 中，共定義了：

### 1 個 Server

```yaml
name: server1
type: server
```

一般而言，Server 的名稱應該使用：

**FQDN（Fully Qualified Domain Name，完整網域名稱）**

如此才能讓其他參與者無論位於何處，都能透過網路正確連線到 Server。

當然，也可以使用整個系統中已知的 Hostname。

---

### 2 個 Client

```yaml
site-1
site-2
```

它們都是：

```yaml
type: client
```

代表兩個參與聯邦學習訓練的站點。

---

### 1 個 Admin

```yaml
admin@nvidia.com
```

其角色設定為：

```yaml
role: project_admin
```

`project_admin` 擁有整個專案中最高的管理權限。

值得注意的是：

**每個專案只能有一位 `project_admin`。**

---

**Builders**

在 NVIDIA FLARE 的 Provisioning 系統中，**Builder** 是一系列 Python 類別（Python Classes），它們會根據 `project.yml` 中的設定，共同產生所有參與者所需的 **Startup Kit**。

Builder 負責建立 Startup Kit 的各個部分，例如：

* 工作空間（Workspace）結構
* 設定檔（Configuration Files）
* 安全憑證（Security Credentials）

在 `builders` 區段中列出的 Builder，會依照**由上而下的順序**依次執行。

本範例使用了幾個常見的 Builder：

* **`WorkspaceBuilder`**

  建立基本的 Workspace（工作空間）目錄結構。

---

* **`StaticFileBuilder`**

  將固定（Static）的檔案複製到 Workspace 中。

---

* **`CertBuilder`**

  產生安全通訊所需的憑證（Certificates）與金鑰（Keys）。

---

* **`SignatureBuilder`**

  建立數位簽章（Signature），以確保檔案未被竄改（tamper-proof）。

---

開發者也可以自行撰寫 Custom Builder，以產生更符合需求的 Provisioning 結果。

本課程不深入介紹 Builder 的運作方式與客製化方法。

目前最重要的是了解：

> **Builder 會依照 `builders` 區段所列出的順序執行，共同完成整個 Provisioning 流程，並產生所有參與者所需的 Startup Kit。**

可以看到，**Provisioning** 完成後，會產生一個具有明確結構的目錄階層（directory hierarchy）。

最上層是一個以 **聯邦學習專案名稱**命名的根目錄（root folder），本範例中為：

```text
example_project
```

在 `example_project` 之下，可以看到多個子目錄：

* **`resources`**

  此目錄用來存放整個專案可能共用的資源（shared resources）或其他額外需要的檔案。

---

* **`state`**

  此目錄用來保存 Provisioning 流程的狀態資訊（state information），或記錄目前各個參與者（participants）的相關狀態。

---

* **`prod_NN`**

  此目錄包含一次成功執行 Provisioning 後，為所有參與者產生的 **Startup Kit（啟動套件）**。

  其中：

```text
NN
```

是一個遞增編號，每成功執行一次 Provisioning，就會增加一次，用來區分不同的 Provisioning 結果。

例如：

```text
prod_00
prod_01
prod_02
...
```

在本範例中，由於是第一次執行 Provisioning，因此產生的是：

```text
prod_00
```

---

**Startup Kit（啟動套件）** 是 Provisioning 為聯邦學習專案中**每一位參與者**所產生的一個資料夾，其中包含執行該參與者所需的檔案、腳本（scripts）與安全憑證（credentials）。

每個 Startup Kit 都只屬於**特定的一位參與者**，通常包含：

* 身分驗證憑證（Authentication Credentials）
* 授權政策（Authorization Policies）
* 防止檔案遭竄改的數位簽章（Signatures for Tamper-proof Mechanisms）
* 方便啟動該參與者的 Shell Script

---

在本範例中，可以看到總共產生了 **4 個 Startup Kit**，分別對應：

* 1 個 Server
* 2 個 Client
* 1 個 Admin

每個資料夾都以其對應參與者在設定檔中所定義的名稱命名。

---

接下來，我們將進一步查看 Server、Client 與 Admin 所產生的 Startup Kit 內容。

由於 **Server 與 Client 的 Startup Kit 結構非常相似**，因此接下來只示範 Server 的 Startup Kit 檔案內容，並透過下列指令進行查看。

各個子資料夾的用途如下：

* **`local` 子資料夾**

  此資料夾包含與**站點（Site）本地設定**相關的組態檔（configuration files），例如：

  * 授權（Authorization）政策
  * 日誌（Logging）設定
  * 隱私（Privacy）設定
  * 資源存取（Resource Access）政策

  每個站點都可以自行修改這些設定檔，以建立符合自身需求的本地政策（local policies）。

  我們將在下一章更詳細介紹**本地站點政策管理（Local Site Policy Management）**。

---

* **`startup` 子資料夾**

  此資料夾包含參與者啟動或離開聯邦學習專案所需的 Shell Script：

  * `start.sh`

    用於啟動參與者，使其加入聯邦學習專案。

  * `stop_fl.sh`

    用於停止執行，使參與者離開聯邦學習專案。

  此外，`startup` 目錄中還包含：

  * 憑證（Certificates）
  * 數位簽章（Signatures）
  * 金鑰（Keys）

  這些檔案是建立各個參與者之間**安全通訊（Secure Communication）**所必需的。

  值得注意的是，憑證與簽章檔案是確保參與者身分可信與資料完整性的核心機制。

  **Provisioning 完成後，不建議手動修改這些檔案**；否則可能導致該參與者無法成功連線到 FL 系統。

---

* **`transfer` 子資料夾**

  此資料夾用於在專案執行期間（runtime）存放各種需要傳輸的產物（artifacts），例如：

  * 自訂應用程式（Custom Application）
  * 腳本（Scripts）
  * 其他檔案（Files）

  這些檔案會在專案運行過程中，由系統進行傳輸或部署。

---

接下來，我們來看看 **`admin` Startup Kit** 的內容。

`admin` 的 **Startup Kit** 具有與 Server、Client 類似的目錄結構，但用途略有不同。

* **`local` 子資料夾**

  `local` 資料夾是空的。

  原因是 `admin` 屬於**管理專案的人員（human participant）**，而不是執行 FL 應用程式的運算站點，因此不需要設定站點層級的本地政策（local policies）。

---

* **`startup` 子資料夾**

  除了憑證（Certificates）與金鑰（Keys）之外，`startup` 資料夾中還包含：

  ```text
  fl_admin.sh
  ```

  這個 Shell Script 可讓 `admin` 使用者登入 **FLARE Console**。

  透過 **FLARE Console**，管理者可以在任何具有安全網路連線的地方，執行各種專案管理工作，例如：

  * 管理聯邦學習專案
  * 監控專案執行情況
  * 提交或管理工作（Jobs）

  後續章節將會介紹 **FLARE Console** 的使用方式。

---

* **`transfer` 子資料夾**

  在 `admin` 的 Startup Kit 中，`transfer` 是預設的執行期（runtime）資料存放位置。

  例如，它可以儲存：

  * 聯邦學習工作（Federated Jobs）完成後，從 Server Workspace 下載回來的結果或相關檔案。

  此外，它也可以用來：

  * 放置（hold）聯邦學習應用程式
  * 或建立連結（link）到使用 FLARE 開發的聯邦學習應用程式

  如此一來，`admin` 使用者便可以方便地將這些應用程式提交（submit）到 Server 執行。

  我們稍後將會更詳細介紹這個流程的運作方式。

**3. 將 Startup Kit 分發給各個參與者（Distribute the startup kits to participants）**

當 Startup Kit 產生完成後，Provisioning 流程的最後一步，就是將這些 Startup Kit 分發給對應的參與者（participants）。

可以使用各種方式進行分發，例如：

* Email
* SFTP（Secure File Transfer Protocol）
* 或其他安全的檔案傳輸方式

重點是必須確保整個傳輸過程具有足夠的安全性。

---

一般而言，每個站點（Site）都會有一位**組織管理員（Organization Admin）**負責接收或下載 Startup Kit。

取得 Startup Kit 之後，組織管理員可以進一步完成該站點的部署工作，例如：

* 安裝所需的套件（Packages）
* 啟動相關服務（Services）
* 設定資料儲存位置（Map the data location）
* 配置授權政策（Authorization Policies）
* 設定組織層級的站點隱私政策（Organization-level Site Privacy Policies）

---

除了自行透過 Email、SFTP 等方式分發 Startup Kit 外，也可以使用 **FLARE Dashboard**。

FLARE Dashboard 提供方便的 **Web UI（網頁介面）**，可用來管理與分發 Startup Kit。

本課程將於下一章簡要介紹 FLARE Dashboard。

---

至此，我們已經了解如何透過 **NVIDIA FLARE 的 Provisioning 流程**，正確地建立一個聯邦學習專案，並為所有參與者產生對應的 **Startup Kit**。

**Proof-of-Concept（PoC）模式**

現在，讓我們把前面學到的 **FLARE Provisioning** 實際應用起來，執行一個已經完成 Provisioning 的聯邦學習應用程式，其中包含：

* Server
* Client
* Admin

等所有參與者。

---

雖然這聽起來很令人期待，但在一般情況下，這並不是一件簡單的工作。

因為我們必須分別為：

* Server
* Client
* Admin

建立各自的分散式運算環境（distributed computing environment）。

如果只是想測試 FLARE 的功能，這樣的部署成本就顯得相當高。

因此，NVIDIA FLARE 提供了 **Proof-of-Concept（PoC）模式**。

PoC 模式允許使用者**在單一電腦上**測試 FLARE 的 Provisioning 與 Deployment 功能，而不需要真正建立一個分散式環境，也不需要配置 Server 與 Client 之間的安全通訊機制。

---

PoC 模式與 **FL Simulator** 有所不同。

### FL Simulator

在 FL Simulator 中：

* 所有工作（Job）的執行都由系統自動完成。
* 整個流程在同一個系統內執行。
* 使用者幾乎不需要管理 Server 或 Client。

可以理解成：

```text
          FL Simulator
                │
      自動管理整個聯邦流程
                │
      Server + Client + Job
```

主要用途是：

* 快速開發
* 快速測試演算法
* Prototype（原型開發）

---

### PoC Mode

PoC 模式則不同。

它允許：

* Server
* Client
* Admin

以**不同的 Process（程序）**分別啟動，再彼此建立連線。

也就是說，即使它們都運行在同一台電腦上，仍然具有較接近真實部署的架構：

```text
          Server Process
                │
      ┌─────────┴─────────┐
      │                   │
 Client Process      Client Process
      │
      │
 Admin Process
```

因此，它比 FL Simulator 更貼近實際部署情境。

---

PoC 模式也讓 **Admin** 能夠透過：

* **FLARE Console**
* **FLARE API**

來：

* 協調（orchestrate）整個專案
* 管理（manage）聯邦學習工作

因此，在正式部署到真實環境之前，PoC 模式是一個非常實用的測試工具。

---

一般而言，開發流程通常如下：

```text
集中式程式
        │
        ▼
改寫成 FLARE Client
        │
        ▼
FL Simulator
（驗證演算法）
        │
        ▼
PoC Mode
（驗證 Server/Client/Admin）
        │
        ▼
Provisioning
        │
        ▼
Production Deployment
```

也就是說，開發者通常會先利用 **PoC 模式**在本機測試應用程式，確認整體流程沒有問題，再部署到正式環境。

---

從 PoC 模式轉換到正式部署時，也意味著系統將從一個較簡化的測試環境，進入具有完整安全機制的環境，其中包括：

* 雙向身分驗證（Mutual Authentication）
* 授權政策（Authorization Policies）

等安全功能。

---

接下來，我們將介紹 **NVFlare CLI** 中 PoC 模式的使用方式。

`nvflare poc prepare`

這是在啟動 **PoC 模式專案**時，第一個需要執行的指令。

此指令會產生一個本機專案工作空間（local project workspace），並為所有參與者建立對應的 **Startup Kit**。

在內部運作上，它其實會執行前一節介紹過的 **Provisioning** 流程。這個流程可以使用：

* 使用者指定的設定檔
* 或預設設定檔

來產生專案與參與者所需的檔案。

以下是 `nvflare poc prepare` 的說明資訊：


## Chapter 4：進階主題、應用案例與延伸學習資源（Advanced Topics, Use Cases and Additional Learning Resources）

本 Notebook 將介紹 NVIDIA FLARE 在實際聯邦學習部署中的進階功能，包括：

* 隱私保護技術（Privacy-preserving Technologies）等安全機制
* Site Policy（站點政策）管理
* 對**機密運算（Confidential Computing）**的支援
* **FLARE Dashboard** 網頁管理介面
* 其他部署方式（Deployment Options）
* **Flower-on-FLARE** 的整合使用

此外，也會簡要介紹：

* 聯邦學習的典型應用案例（Use Cases）
* 採用 NVIDIA FLARE 的真實世界聯邦學習專案

最後，將提供更多延伸學習資源，協助希望深入了解 **NVIDIA FLARE 實際部署與開發**的開發者持續學習。
