# vitaldb-htn-risk-analysis _ 🩺 VitalDB 기반 수술 전 고혈압 위험요인 분석  
### EDA · Statistical Test · Logistic Regression · ML Modeling · SHAP · Clustering (UMAP-Gower)

---

## 📌 1. 프로젝트 개요 (Project Overview)

본 프로젝트는 **VitalDB(서울대병원 개방 의료 데이터셋)**를 활용하여  
수술 전 환자의 임상적 특성을 기반으로 **고혈압(preop_htn) 발생과 관련된 요인 분석 및 예측 모델을 구축**하는 것을 목표로 한다.

본 분석은 다음 4가지 축으로 이루어진 **종합 데이터사이언스 프로젝트**이다:

1. **Cohort-level EDA (기초군 분석)**  
2. **임상 변수 간의 Group Comparison + 통계 검정**  
3. **ML 기반 예측 모델링 + 설명가능성(SHAP)**  
4. **비지도 학습 기반 환자군 패턴 탐색 (Gower/UMAP + Clustering)**

본 프로젝트는 강의 과제를 넘어서,  
**실제 임상 연구로 확장 가능한 구조**를 목표로 설계되었다.

---

## 📁 2. 데이터 구성 (VitalDB)

본 프로젝트에서는 VitalDB의 다음 데이터를 활용하였다:

- `cases.csv`  
  - 환자 단위 메타데이터  
  - ASA, 나이, BMI, 수술 부위, 수술 시간, 검사 수치 등 포함

- ECG 신호 데이터는 본 분석에서 **직접 타겟으로 사용하지 않음**  
  (대신 HTN과 window-level HRV 기반 분석은 별도 참고)

---

## 🧹 3. 데이터 전처리 (Preprocessing)

### ✔ 주요 처리 단계

- 성인(age ≥18) 환자만 포함  
- preop_htn ∈ {0,1}  
- 수술 유형(optype), department 등 범주형 정제  
- 수술 시간 outlier 처리 (IQR 기반 trimming)  
- 클러스터링 전용 변수 결측치 처리  
  - numeric → median  
  - categorical → mode  

---

## 📊 4. EDA & 통계 분석

### ✔ EDA 결과 요약
- HTN 비율: **약 31%**
- 고혈압군은  
  - 나이 ↑  
  - BMI ↑  
  - ASA ↑  
  - PT·APTT·Glucose ↑  
  등의 일관된 특징을 가짐

### ✔ 통계 검정
- numeric → Mann-Whitney U  
- categorical → Chi-square  
- ASA / optype / labs 대부분 유의미(p < 0.001)

---

## 📈 5. 로지스틱 회귀 (Logistic Regression)

- 종속 변수: `preop_htn`
- 독립 변수: ASA, age, BMI, labs 등

### ✔ 주요 Odds Ratio
| Feature | OR | Interpretation |
|--------|--------|----------------|
| ASA 3 | ~14.8 | 고혈압 위험 대폭 증가 |
| ASA 2 | ~7.5 | 고혈압 위험 증가 |
| BMI | ~1.14 per unit | 체중 증가 영향 |
| Age | ~1.06 per unit | 연령 증가 영향 |
| PT/APTT/Glucose | 유의미한 증가 |

임상적으로도 일관적인 결과.

---

## 🤖 6. 머신러닝 기반 HTN 예측 모델

### 사용 모델
- Logistic Regression  
- RandomForest  
- GradientBoosting (best)  
- SVM (RBF)

### ✔ 최종 성능 (5-Fold CV)
| Model | AUC |
|-------|-------|
| **GradientBoosting** | **0.812** |
| RandomForest | 0.806 |
| SVM | 0.803 |
| LogisticRegression | 0.800 |

### ✔ Test ROC-AUC  
`0.8027`

---

## 🔍 7. SHAP 기반 Explainability

### ✔ Global Feature Importance  
1) ASA  
2) Age  
3) BMI  
4) PT / Glucose  
5) Preop Hemoglobin  

### ✔ Dependence Plot 분석
- **ASA**: 강한 양의 영향  
- **BMI**: 선형 증가 패턴  
- **Glucose**: 대사성 위험군과 연관  
- **PT/APTT**: 경미하지만 유의미한 영향  

---

## 🔮 8. 비지도 클러스터링 (Unsupervised Clustering)

### ✔ 방법 A: Gower distance + Hierarchical clustering  
- 최적 cluster = **2개**
- 저위험군 vs 고위험군 형태의 자연스러운 분리

### ✔ 방법 B: UMAP → KMeans  
- 비선형 구조 반영  
- k=3~4에서도 실질적 분리 가능  
- UMAP 2D 시각화로 분포 확인

---

## 🔥 9. 클러스터별 SHAP 비교 (핵심)

cluster별로 SHAP을 재계산하여  
“고혈압 위험이 어떤 요인 때문에 발생하는지”  
군집 간 차이를 분석함.

예시:
- Cluster 0: BMI, 성별 등 생활 습관 요인 비중 ↑  
- Cluster 1: ASA, age 영향 압도적으로 ↑  
- Cluster 2: Glucose, PT 등 대사/응고 요인 중심 ↑

이는  
> “고혈압이라는 단일 label이 실제로는 서로 다른 기전에 의해 발생”  
한다는 것을 보여주는 중요한 결과.

---

## 📦 10. 프로젝트 구조

