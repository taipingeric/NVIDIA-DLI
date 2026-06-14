# 第三章介紹（Introduction）

歡迎來到 **NVIDIA FLARE 自主學習課程**的第三章！

在本章中，我們將介紹 **NVIDIA FLARE（NVFlare）** 的核心概念與系統架構，了解它如何作為一個強大的**聯邦運算（Federated Computing）**平台。我們將探討 NVFlare 系統的各個組成元件、學習如何模擬部署環境，以及了解與系統互動的不同方式。

---

# 學習目標（Learning Objectives）

完成本章後，你將能夠：

* 區分**聯邦學習（Federated Learning）**與**聯邦運算（Federated Computing）**的差異，並了解 NVIDIA FLARE 在其中所扮演的角色。
* 辨識並說明 NVIDIA FLARE 架構中的核心元件。
* 理解這些元件如何彼此協作，以完成聯邦運算工作流程。

---

# 聯邦學習與聯邦運算：了解兩者的差異

## Federated Learning vs. Federated Computing

在深入技術細節之前，先釐清一個重要觀念：

### 聯邦學習（Federated Learning）

聯邦學習是一種**分散式機器學習（Distributed Machine Learning）**的方法。

在此架構下，模型會在多個持有本地資料的裝置或伺服器上進行訓練，而**資料本身不需要離開原始位置，也不需要彼此交換**。

換句話說：

> **模型移動，資料不移動。**

---

### 聯邦運算（Federated Computing）

聯邦運算則是一個**更廣泛的概念**。

它提供一套架構，使不同地點的系統可以共同完成運算，同時維持：

* 資料隱私（Privacy）
* 資料安全（Security）
* 分散式協作（Distributed Collaboration）

而聯邦學習只是建立在聯邦運算之上的其中一種應用。

---

## NVIDIA FLARE 的定位

NVIDIA FLARE 是一個**聯邦運算框架（Federated Computing Framework）**。

在此框架之上，可以建構許多不同應用，例如：

* Federated Learning（聯邦學習）
* Federated Analytics（聯邦分析）

等各種分散式資料運算工作。

其主要特色包括：

### • 不受資料型態限制（Data Domain Agnostic）

可支援任何資料型態、工作流程或應用領域，而不限於特定格式。

---

### • 將運算帶到資料所在地（Computation at the Data）

不同於 Data Lake 等集中式架構，需要先將資料集中，

NVIDIA FLARE 的理念是：

> **把運算能力帶到資料所在地，而不是把資料搬到運算中心。**

---

### • 保護資料隱私（Privacy-Preserving）

資料始終保留在原始節點中。

合作單位之間只交換經過授權的結果，例如：

* 模型權重
* 統計資訊
* 指定分析結果

而不分享原始資料。

---

### • 不受系統限制（System Agnostic）

透過 FLARE Client，可輕鬆整合各種：

* 資料處理框架
* Machine Learning Framework
* Deep Learning Framework

因此具有高度相容性。

---

### • 彈性的部署方式（Deployment Flexibility）

可部署於多種環境，例如：

* Sub-process
* Docker Container
* Kubernetes Pod
* HPC（High Performance Computing）
* 其他特殊運算系統

---

# NVIDIA FLARE 的核心概念

## Core Concepts of NVIDIA FLARE

要了解 NVIDIA FLARE 的運作方式，需要先認識其架構中的幾個核心概念。

---

# 1. Controller：協調者（Orchestrator）

## 什麼是 Controller？

Controller 定義整個聯邦運算流程，並負責協調所有 Client 的工作。

它可以視為整個系統的「指揮中心」。

---

## 主要職責

* 決定聯邦運算如何執行
* 定義協調模式（例如 Round-robin、Scatter & Gather）
* 將任務分派給 Client
* 接收並處理 Client 回傳的結果

---

## Controller 執行位置

大多數情況下：

Controller 運行於 FL Server，因此也稱為 **Server Strategy**。

但也可以部署在 Client 端（Client-side Controller），例如：

* Swarm Learning
* Peer-to-Peer Workflow

等去中心化架構。

---

## 為什麼重要？

Controller 是聯邦系統的大腦，它決定：

* 哪些 Client 參與
* 每個 Client 執行什麼任務
* 如何整合所有結果

---

# 2. Executor：執行者（Worker）

## 什麼是 Executor？

Executor 是執行於 Client 端的邏輯元件。

它負責實際完成 Controller 指派的工作。

---

## 主要職責

* 接收 Controller 任務
* 執行本地運算（例如模型訓練、模型評估）
* 將結果回傳 Controller
* 管理本地資料與運算資源

---

## 為什麼重要？

真正的運算工作都是由 Executor 完成。

它直接使用本地資料進行計算，而**原始資料不會被分享出去**。

因此，

> **Controller 負責指揮，Executor 負責執行。**

兩者共同構成 NVIDIA FLARE 聯邦運算架構的核心。

---

# 3. Shareable：通訊媒介（Communication Medium）

## 什麼是 Shareable？

Shareable 是 Server 與 Client 之間交換資訊的標準化物件。

所有資料都透過 Shareable 傳遞。

---

## 技術實作

本質上，

Shareable 是一個 Python Dictionary，包含兩大部分：

### Header

包含：

* Peer Properties（傳送者資訊）
* Cookie（可雙向傳遞的狀態資訊）
* Return Code（執行狀態）

---

### Content

真正要傳送的資料，例如：

* 模型權重
* 評估指標
* 統計資訊

---

## 為什麼重要？

Shareable 提供統一的通訊格式，使系統更容易：

* 實作安全機制
* 追蹤資料來源（Provenance）
* 維持元件相容性

---

## 範例

Server 將模型送至 Client：

```
Server
      │
      │ Shareable(Model)
      ▼
Client
      │
      │ Training
      ▼
Client
      │
      │ Shareable(Updated Model)
      ▼
Server
```

模型權重會包裝成 Shareable 傳送；

完成訓練後，再將更新後模型包裝成 Shareable 回傳。

---

# 4. Filter：守門員（Gatekeeper）

## 什麼是 Filter？

Filter 可以在 Shareable 傳送或接收時，對資料進行轉換。

---

## 功能

可以：

* 檢查資料
* 修改資料
* 壓縮資料
* 加密資料
* 紀錄資料

---

## 可實作功能

例如：

* Differential Privacy（差分隱私）
* Model Compression（模型壓縮）
* Encryption（加密）
* Security Policy Validation（安全政策驗證）
* Logging（紀錄）

---

## 為什麼重要？

Filter 提供模組化方式，在**不修改核心通訊流程**的情況下，就能增加：

* 隱私保護
* 安全性
* 傳輸效率

---

## 使用範例

例如：

* 在模型更新加入雜訊以實現 Differential Privacy
* 壓縮模型權重降低網路頻寬
* 加密敏感資訊
* 驗證分享內容是否符合隱私政策

Filter 可以配置於資料傳送端與接收端，因此形成一條彈性的資料處理流程。

---

# 更高層級的抽象（Higher-Level Abstractions）

除了上述核心概念之外，NVIDIA FLARE 也提供更高層級的抽象介面，方便資料科學家與機器學習開發者使用。

## FLModel：適合資料科學家的資料結構

FLModel 是專為聯邦學習設計的標準化資料結構。

其主要內容包括：

* **params_type**：參數類型（例如 `FULL`、`DIFF`）
* **params**：模型參數（例如神經網路權重）
* **optimizer_params**：Optimizer 參數
* **metrics**：評估指標（Loss、Accuracy 等）
* **round information**

  * `start_round`
  * `current_round`
  * `total_rounds`
* **meta**：額外 Metadata（Key-Value Dictionary）

### 幕後運作方式

FLModel 會自動與 Shareable 相互轉換，因此開發者不需處理底層通訊細節。

### 為什麼重要？

FLModel 建立了 NVIDIA FLARE 與外部訓練系統之間的統一介面，使開發者能更專注於模型本身，而非資料交換流程。

---

# 整體運作流程（Putting It All Together）

綜合上述概念，NVIDIA FLARE 的運作流程如下：

1. **Controller（通常位於 Server）**負責協調整個聯邦工作流程。

2. Controller 透過 **Shareable** 與各 Client 的 **Executor** 交換資訊。

3. **Filter** 可在資料傳輸過程中加入：

   * 隱私保護
   * 安全機制
   * 傳輸效率最佳化

4. 所有元件皆建立於 **FLComponent** 基底類別，因此形成事件驅動（Event-driven）架構。

5. 各元件透過 **FLContext** 與 **Event** 機制互相溝通。

6. **FLModel** 等高階抽象介面，讓特定應用開發更加容易。

下一節將更深入介紹 **System Architecture**，了解這些概念如何在實際系統中運作。

---

# 本章重點（Key Takeaways）

* NVIDIA FLARE 是一個聯邦運算平台，其核心理念是**將運算帶到資料所在地，而非集中資料**。
* **Controller 與 Executor** 的互動構成聯邦工作流程的核心。
* **Shareable** 提供標準化的元件通訊格式。
* **Filter** 以模組化方式實現隱私保護、安全性與效率提升。
* 基於 **FLComponent** 的事件驅動架構，使系統具有高度彈性與可擴充性。
* **FLModel** 等高階抽象介面，大幅簡化聯邦學習應用程式的開發。
