# 第四章：聯邦系統的建置與部署

本章將介紹如何建立一套可用於**模擬環境（Simulated Deployment）**或**實際環境（Real-world Deployment）**的聯邦學習系統。我們將學習 NVIDIA FLARE 的系統建置（Provisioning）流程，以及如何在 Docker、雲端平台與 Kubernetes 等不同環境中部署聯邦學習系統。

本章將涵蓋以下主題：

## 1. NVIDIA FLARE 的 Provisioning

Provisioning 是建立聯邦學習系統的重要步驟，負責產生 Server、Client 等節點所需的設定檔、憑證及部署資源。

* **NVIDIA FLARE Provisioning 簡介**

  * [Introduction to Provisioning in NVIDIA FLARE](provision.ipynb)

* **使用 `nvflare provision` 指令進行 Provisioning**

  * [Provision via `nvflare provision` CLI](../04.1_provision_via_cli/provision_via_cli.ipynb)

* **透過 FLARE Dashboard 進行 Provisioning**

  * [Provision via FLARE Dashboard](../04.2_provision_via_dashboard/provision_via_dashboard.ipynb)

---

## 2. Preflight Check（部署前檢查）

在正式部署聯邦系統之前，需要先檢查執行環境是否符合需求，例如網路設定、節點配置及必要資源，以確保部署過程順利。

* [Preflight Check](../04.3_preflight_check/preflight_check.ipynb)

---

## 3. 容器化部署（Containerized Deployment）

介紹如何利用 **Docker** 建立並執行 NVIDIA FLARE 聯邦學習系統，讓部署流程更加標準化且容易管理。

* [使用 Docker 進行 Provision 與執行](../04.4_provision_and_run_with_docker/provision_and_run_with_dockers.ipynb)

---

## 4. 雲端部署（Cloud Deployment）

介紹如何將 NVIDIA FLARE 部署至主流雲端平台，包括：

* [在 AWS 上部署 NVIDIA FLARE](../04.5_deployment_in_aws/deployment_in_aws.ipynb)

* [在 Azure 上部署 NVIDIA FLARE](../04.6_deployment_in_azure/deployment_in_azure.ipynb)

---

## 5. Kubernetes 部署

介紹如何利用 **Kubernetes（K8s）** 建立可擴充且易於管理的聯邦學習叢集。

* [使用 Kubernetes（K8s）進行 Provision 與執行](../04.7_provision_and_run_with_k8s/provision_and_run_with_k8s.ipynb)

---

## 6. 本章回顧（Recap）

整理本章所學的重要概念與部署流程。

* [本章回顧](../04.8_recap/recap.ipynb)

---

# 開始學習！

接下來，讓我們先從 **NVIDIA FLARE Provisioning 簡介**開始，了解如何建立聯邦學習系統所需的基本設定與部署流程。
