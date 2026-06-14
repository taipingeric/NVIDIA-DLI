# 第六章：聯邦學習系統的安全機制（Security in Federated Learning Systems）

聯邦學習（Federated Learning, FL）雖然避免了集中式資料收集，但仍面臨許多來自模型、系統、通訊以及參與者層面的安全與隱私挑戰。因此，一個完整的聯邦學習系統必須建立完善的安全架構，以確保資料、模型與參與者皆受到保護。

---

# 聯邦學習系統的重要安全議題

## Critical Security Concerns in Federated Learning System

### 1. 資料隱私（Data Privacy）

即使原始資料不離開本地端，仍可能透過模型更新資訊洩漏敏感內容，因此需要防範下列攻擊：

* **Model Inversion Attack（模型反演攻擊）**

  * 從模型參數反推出訓練資料內容。

* **Membership Inference Attack（成員推論攻擊）**

  * 判斷某筆資料是否曾參與模型訓練。

* **Property Inference Attack（屬性推論攻擊）**

  * 推測訓練資料具有哪些共同特性。

* **Gradient Leakage（梯度洩漏）**

  * 在模型梯度交換過程中推導出原始資料。

---

### 2. 系統安全（System Security）

聯邦系統由多個組織共同參與，因此需要確保整體系統安全，包括：

* 參與者身分驗證（Authentication）
* 中間人攻擊（Man-in-the-Middle Attack）
* Sybil Attack（惡意建立多個假身分）
* 阻斷服務攻擊（DoS Attack）
* 模型與梯度傳輸過程中的網路安全

---

### 3. 模型安全（Model Security）

除了保護資料，也必須保護模型本身，例如避免：

* Model Poisoning Attack（模型污染攻擊）
* Backdoor Attack（後門攻擊）
* Model Stealing／Extraction（模型竊取）
* Adversarial Attack（對抗攻擊）

---

### 4. 參與者隱私（Participant Privacy）

需要保護參與聯邦學習的組織與人員資訊，包括：

* 參與者身分保護
* 是否參與聯邦學習計畫的保密性
* 組織智慧財產（Intellectual Property）的保護

---

### 5. 計算完整性（Computation Integrity）

系統必須確認每個參與者確實執行正確的運算，例如：

* 驗證計算是否正確
* 偵測惡意或異常更新
* 確保各方遵循聯邦學習協定

---

### 6. 存取控制（Access Control）

不同角色應具有不同權限，例如：

* 角色式權限管理（Role-Based Access Control）
* 資源使用限制
* 模型存取權限
* 資料存取限制

---

### 7. 法規遵循（Regulatory Compliance）

聯邦學習亦須符合相關法規，例如：

* GDPR
* HIPAA
* 跨境資料治理（Cross-border Data Governance）
* 稽核紀錄（Audit Trail）與責任追蹤（Accountability）

---

### 8. 基礎設施安全（Infrastructure Security）

包含整體運算環境的安全性：

* Edge Device 安全
* Server 安全
* 通訊通道安全
* 模型檢查點（Checkpoint）儲存安全

---

### 9. 信任管理（Trust Management）

建立參與者彼此之間的信任，例如：

* Reputation System（信譽機制）
* 信任建立機制
* 驗證參與者是否合法

---

### 10. 聚合安全（Aggregation Security）

模型聚合過程也需要安全機制，例如：

* Secure Aggregation Protocol
* 防止參與者串通（Collusion）
* Byzantine-Robust Aggregation（容錯聚合）

---

# 聯邦學習系統的安全機制

## Security Mechanisms in Federated Learning System

聯邦運算系統需要完善的安全機制，以確保：

* 只有合法且可信任的參與者可以加入
* 通訊內容受到保護
* 權限受到適當管理

以下介紹幾項重要安全元件。

---

## Authentication（身分驗證）

Authentication 用於確認通訊雙方的身份：

> **確保每個人都是其宣稱的身份。**

例如：

* Server 驗證 Client
* Client 驗證 Server

避免未授權節點加入系統。

---

## Authorization（授權）

Authorization 用於確認：

> **使用者只能執行其被允許的操作。**

例如：

* 是否可以提交 Job
* 是否可以下載模型
* 是否可以管理 Client

由於聯邦學習涉及多個組織，因此每個組織都需要自己的 Authentication 與 Authorization 機制。

NVIDIA FLARE 透過 **事件驅動（Event-based）的 Federated Authentication 與 Authorization** 來實現此功能。

---

## Privacy Protection（隱私保護）

聯邦學習最大的目標之一，就是在不分享原始資料的情況下共同訓練模型。

然而模型更新仍可能洩漏資訊，因此需要各種隱私保護技術（Privacy Enhancing Technologies, PETs）。

第五章已介紹相關 PET 技術，本章則著重於**組織層級（Organization Level）**的隱私保護機制。

---

## Trust-based Security（基於信任的安全機制）

除了傳統安全機制外，NVIDIA FLARE 也利用：

> **Confidential Computing（機密運算）**

中的

> **Trusted Execution Environment（TEE）**

建立可信任執行環境。

未來 NVIDIA FLARE 將支援：

> **End-to-End Confidential Federated AI**

提供更完整的保護。

本章僅做簡介，後續版本將有更深入介紹。

---

## Communication Security（通訊安全）

NVIDIA FLARE 使用安全通訊協定保護資料傳輸：

* TLS（Transport Layer Security）
* Mutual TLS（mTLS）
* TLS 搭配 Signed Messages

以確保：

* 傳輸加密
* 身分驗證
* 訊息完整性

---

# NVIDIA FLARE 安全架構

## NVIDIA FLARE Security Architecture

NVFLARE 是部署於各參與組織 IT 環境中的應用程式。

因此整體安全性來自兩部分：

1. NVFLARE 本身提供的安全機制
2. 各組織 IT 基礎設施提供的安全措施

---

NVFLARE 主要涵蓋以下安全面向：

## Identity Security

負責：

* Authentication
* Authorization

確保通訊雙方身份合法。

---

## Site Policy Management

每個組織可自行制定：

* Resource Management
* Authorization Policy
* Privacy Policy

因此不同機構可擁有不同管理策略。

---

## Communication Security

保護所有通訊訊息的：

* Confidentiality
* Integrity

避免竊聽與竄改。

---

## Message Serialization

提供安全的：

* Serialization
* Deserialization

避免惡意資料造成攻擊。

---

## Data Privacy Protection

利用各種技術避免：

* Local Data Leakage
* Reverse Engineering

保護本地資料安全。

---

除了上述內容之外，其餘安全議題仍需依賴各組織自身 IT 基礎設施負責，例如：

* 實體安全（Physical Security）
* Server Security
* Client Machine Security

此外，在可信環境中，TLS 亦提供身份驗證機制。

---

# 名詞介紹（Terminologies）

NVIDIA FLARE 使用以下術語：

## Project

一個具有明確參與者的聯邦學習研究計畫。

---

## Org（Organization）

參與聯邦學習計畫的組織。

---

## Site

執行 NVIDIA FLARE 的運算節點。

Site 分為兩種類型：

* Server Site
* Client Site

每個 Site 都隸屬於某個 Organization。

---

## FL Server

部署於 Server Site 的應用程式，

負責：

* 協調 Client
* 執行 Federation Workflow

---

## FL Client

部署於 Client Site 的應用程式，

負責：

* 接收 Server 任務
* 使用本地資料完成訓練
* 回傳結果

---

## User

參與聯邦學習計畫的人員。

---

# 角色（Roles）

每位使用者都會被指定一種角色，不同角色具有不同權限。

共有四種角色：

---

## Project Admin

負責：

* 建立 Project
* Provision 所有參與者
* 協調所有組織

每個 Project 僅有一位 Project Admin。

---

## Org Admin

負責管理自己組織內的：

* Sites
* 人員
* 系統

---

## Lead Researcher

組織中的主要研究人員，

通常具有較高權限，

負責與其他研究人員合作完成計畫。

---

## Member Researcher

一般研究人員，

權限較低，

協助 Lead Researcher 完成各項工作。

---

## FLARE Console

執行於使用者電腦上的命令列工具，

可用來操作 NVFLARE 系統。

---

# NVIDIA FLARE 安全架構

NVIDIA FLARE 採用：

* PKI（Public Key Infrastructure）進行身份驗證
* TLS 保護資料傳輸

並搭配以下安全機制：

* Filter 機制與本地隱私政策
* Federated Authorization（聯邦授權）
* Site-specific Authentication（各 Site 自訂驗證）
* Privacy Algorithms：

  * Differential Privacy（差分隱私）
  * Homomorphic Encryption（同態加密）
  * Multi-party Computing（例如 Private Set Intersection）
* Confidential Computing（機密運算）

---

# 本章內容

本章將依序介紹 NVIDIA FLARE 的各項安全機制：

* **6.1 Identity Security（身份安全）**
* **6.2 Site Security and Privacy Policy（站點安全與隱私政策）**
* **6.3 Customized Site Security（自訂站點安全機制）**
* **6.4 Communication Security（通訊安全）**
* **6.5 Message Serialization（訊息序列化安全）**
* **6.6 Trust-based Security（基於信任的安全機制）**
