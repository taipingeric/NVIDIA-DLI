# 執行聯邦學習應用程式（Running Federated Learning Applications）

本章將介紹如何使用 **NVIDIA FLARE** 執行聯邦學習（Federated Learning）應用程式。我們將從環境建置與資料準備開始，接著使用 **PyTorch** 訓練分類器，並進一步將傳統深度學習模型轉換為聯邦學習架構。此外，我們也會介紹如何客製化伺服器端與客戶端邏輯、設定實驗追蹤（Experiment Tracking），以及說明聯邦學習工作的結構（Job Structure）與相關設定，包括如何執行模擬器（Simulator）。最後，將對本章內容進行總結。

## 1. 執行聯邦學習工作（Running Federated Learning Job）

* [環境建置與準備（Set-up and Preparation）](../01.1_running_federated_learning_job/setup.ipynb)
* [使用 PyTorch 訓練分類器（Training Classifier with PyTorch）](../01.1_running_federated_learning_job/running_pytorch_fl_job.ipynb)

---

## 2. 從單機深度學習到聯邦學習（From Stand-alone Deep Learning to Federated Learning）

* [將使用 PyTorch 的深度學習模型轉換為聯邦學習（Convert Deep Learning with PyTorch to Federated Learning）](../01.2_convert_deep_learning_to_federated_learning/convert_dl_to_fl.ipynb)

---

## 3. 伺服器端客製化（Server-side Customization）

* [客製化伺服器端邏輯（Customize Server Logics）](../01.3_server_side_customization/customize_server_logics.ipynb)

---

## 4. 客戶端客製化（Client-side Customization）

* [客製化客戶端訓練流程（Customize Client Training）](../01.4_client_side_customization/customize_client_training.ipynb)

---

## 5. 訓練指標追蹤（Tracking the Training Metrics）

* [實驗追蹤（Experiment Tracking）](../01.5_experiment_tracking/experiment_tracking.ipynb)

---

## 6. 工作結構與設定（Job Structure and Configurations）

* [聯邦學習工作的結構與設定（Job Structure & Configuration）](../01.6_job_structure_and_configuration/understanding_fl_job.ipynb)

---

## 7. 日誌設定（Logging Configurations）

* [日誌紀錄（Logging）](../01.7_logging/logging.ipynb)