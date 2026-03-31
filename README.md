# 📦 스마트 창고 출고 지연 예측 AI 
> **Smart Warehouse Outbound Delay Prediction Project**

스마트 창고의 효율성을 극대화하기 위해 로봇 상태, 주문량, 레이아웃 등 다양한 변수를 활용하여 **향후 30분 내 발생할 평균 출고 지연 시간**을 시계열 데이터 기반으로 예측하는 프로젝트입니다.

---

## 🚀 1. 프로젝트 개요
* **목표**: 6시간 동안 15분 간격(24 타임스텝)으로 기록된 데이터를 기반으로 다음 30분의 평균 지연 시간($avg\_delay\_minutes\_next\_30m$) 예측
* **데이터 규모**: 약 12,000개 시나리오 × 24 타임스텝 (총 약 288,000 rows)
* **평가 지표**: **RMSE (Root Mean Squared Error)**

---

## 🏗 2. 전체 아키텍처 (3단계 전략)

### **PHASE 1: GBDT Baseline (Local)**
* **환경**: MacBook M5 (Apple Silicon CPU 최적화)
* **모델**: `LightGBM` + `XGBoost` + `CatBoost` Ensemble
* **전략**: `pipeline.py` 기반의 OOF(Out-of-Fold) 예측값 생성 및 앙상블

### **PHASE 2: 피처 엔지니어링 강화 (Local)**
* **시계열 피처**: $t-1, t-2, t-3$ 타임스텝의 Lag 피처 생성
* **이동 통계량**: 3/5 스텝 이동평균(Rolling Mean) 및 표준편차 추가
* **누적 지표**: 시나리오 내 현재 시점까지의 누적 지연 시간 계산
* **교차 피처**: `layout_id`와 시간대(time_step) 결합 피처

### **PHASE 3: 딥러닝 & 스태킹 (Cloud)**
* **환경**: Google Colab T4 (GPU 활용)
* **모델**: `TabNet`, `MLP`, `LSTM`, `Transformer`
* **최종 앙상블**: GBDT와 딥러닝 모델의 OOF를 메타 모델(Meta Learner)로 결합

---

## 💻 3. 환경별 역할 분담

| 환경 | 주요 작업 | 활용 도구 |
| :--- | :--- | :--- |
| **Local (MacBook M5)** | EDA, 피처 엔지니어링, GBDT 모델 학습 | Python, Pandas, Scikit-learn, GBDT Library |
| **Colab T4 (GPU)** | 신경망 기반 딥러닝 모델 학습 | PyTorch, TabNet, TensorFlow |

---

## 📊 4. 핵심 전략: Group K-Fold 및 검증

동일한 시나리오 내의 데이터가 학습셋과 검증셋에 동시에 포함되어 발생하는 **데이터 누수(Data Leakage)**를 방지하기 위해 일반적인 랜덤 분할 대신 **`scenario_id` 기준의 Group K-Fold**를 적용합니다.

* **적용 이유**: 시나리오 내 타임스텝 간의 강한 상관관계를 고려하여 모델의 일반화 성능을 객관적으로 측정하기 위함

---

## 📈 5. 성능 향상 로드맵

| 단계 | 작업 내용 | 예상 RMSE 개선 |
| :--- | :--- | :---: |
| **Step 1** | GBDT Baseline 구축 | 기준점(Base) |
| **Step 2** | 시계열 Lag/Rolling 피처 추가 | -5% ~ 15% |
| **Step 3** | Group K-Fold로 검증 전략 수정 | 신뢰도 향상 |
| **Step 4** | Optuna 하이퍼파라미터 튜닝 | -2% ~ 5% |
| **Step 5** | 딥러닝 모델 결합 (Stacking) | -3% ~ 8% |
