# mclp-smart-shelter-seoul

# 🚌 소방안전 빅데이터를 활용한 서울시 스마트 버스쉘터 최적 입지 선정

> 소방안전 빅데이터 플랫폼 데이터를 기반으로 폭염 취약성과 유동인구를 통합 분석하여,  
> 서울시 스마트 버스쉘터(지능형 버스정류장)의 최적 신규 설치 입지를 도출한 프로젝트입니다.

![Python](https://img.shields.io/badge/Python-3.10-blue?logo=python)
![GeoPandas](https://img.shields.io/badge/GeoPandas-0.14-green)
![Scikit--learn](https://img.shields.io/badge/Scikit--learn-1.3-orange?logo=scikit-learn)
![Folium](https://img.shields.io/badge/Folium-0.15-brightgreen)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

---

## 📌 프로젝트 개요

최근 지구온난화로 폭염 피해가 급증하고 있습니다. 2024년 한 해 온열질환 응급환자는 3,704명(전년 대비 +31.4%), 사망자는 34명에 달했으며, 2025년에는 7월 기준 이미 전년 동기 대비 2배 이상 증가했습니다.

현재 운영 중인 무더위 쉼터는 경로당 등 특정 계층 중심, 제한된 운영시간, 낮은 접근성 등의 구조적 한계를 가지고 있습니다. 이에 본 프로젝트는 **모든 시민이 즉각 이용 가능한 스마트 쉘터**의 신규 설치 우선 입지를 데이터 기반으로 도출하고자 합니다.

| 항목 | 내용 |
|------|------|
| 분석 기간 | 2024년 6월 ~ 8월 (하절기 집중 분석) |
| 분석 시간대 | 낮 12시 ~ 오후 3시 (고온 노출 최대 시간대) |
| 분석 단위 | 서울시 버스정류장 (총 10,376개) |
| 사용 언어 | Python 3.10 |
| 공모 출처 | 소방안전 빅데이터 활용 및 아이디어 경진대회 |

---

## 👤 기여 파트 (본인 담당)

> 팀 프로젝트로 진행되었으며, 본인의 주요 기여 파트는 다음과 같습니다.

- ✅ **문제 정의** — 소방안전 빅데이터 플랫폼 온열질환 구급출동 데이터 분석, 기존 무더위 쉼터의 구조적 한계 도출
- ✅ **데이터 수집** — 유동인구(승차·환승), 기후 노출(ERA5-Land, Landsat 8, S-DoT, 자외선지수), 사회취약성, 적응능력 데이터 수집 및 통합
- ✅ **데이터 전처리** — 후보 정류장 필터링(상위 75%), 로그 변환, StandardScaler 표준화, 좌표계 변환(WGS84 → UTM-K)
- ✅ **VRI 계산** — 노출도(Exposure) / 민감도(Sensitivity) / 적응능력(Adaptive Capacity) 산출 및 통합 VRI 계산, T-score 변환
- ✅ **적응능력(Adaptive Capacity) 산출** — 반경 500m 내 무더위 쉼터 수용인원, 반경 50m 내 녹지면적 지표 구축



---

## 🔬 분석 방법론

### 1. 후보 정류장 필터링
전체 버스정류장 10,376개 중 승차총승객수 · 총환승수 · 가중평균환승시간이 **모두 상위 75%**에 해당하는 정류장 **4,872개**를 분석 대상으로 선별

### 2. VRI (취약성-탄력성 지표) 계산

$$VRI = \frac{\left(\frac{Exposure + Sensitivity}{2}\right) - Adaptive\ Capacity}{2}$$

| 구성요소 | 변수 | 처리 방법 |
|----------|------|-----------|
| **Exposure** | 열대야 일수, 지표면 온도, 자외선 지수 | 표준화 후 평균 |
| **Sensitivity** | 65세 이상 인구비율, 수급자비율, 독거노인비율, 저소득노인비율, 장애인비율, 인구밀도, 주택노후도 | PCA 적용 (PC1: 분산 53.96% 설명) |
| **Adaptive Capacity** | 반경 500m 무더위 쉼터 수용인원, 반경 50m 녹지면적 | 표준화 후 평균 |

> 음수 VRI 값의 MCLP 왜곡을 방지하기 위해 **T-score 변환** 적용 (평균 50, 표준편차 10)

### 3. 가중유동인구 산출

$$가중유동인구 = VRI_{T\text{-}score} \times \log(승차총승객수)$$

### 4. MCLP 최적화

- **서비스 반경**: 400m (보행 접근권 표준)
- **중복 배제**: 기존 스마트쉘터 반경 400m 이내 후보지 사전 제거
- **알고리즘**: Greedy 근사해법
- **목적함수**: 커버되는 가중유동인구 합 최대화

---

## 📊 주요 결과

### MCLP vs Top-K 비교

| 구분 | 커버 면적(㎡) | 수요합 | 면적 증가율 |
|------|-------------|--------|------------|
| 기존 스마트쉘터 | 39.04 | 51,873 | - |
| **MCLP 선정** | **57.29** | **50,268** | **+75.66%** |
| Top-K 선정 | 32.61 | 70,678 | - |

> MCLP는 Top-K 대비 커버 면적 **+75.66%** 향상 — 서비스 사각지대 해소에 효과적

### 우선 설치 추천 정류장 Top 2

| 순위 | 정류장 | 위치 | 특징 |
|------|--------|------|------|
| 🥇 1위 | 서울역버스환승센터 (회현방향) | 서울 중심부 | KTX·지하철·버스 결절점, 높은 유동인구 |
| 🥈 2위 | 수서역환승센터 | 강남권 남단 | GTX·SRT·지하철 3호선 연계, 광역 교통 허브 |

---

## 📦 주요 라이브러리

```
pandas
geopandas
scikit-learn
folium
cdsapi
geemap
scipy
matplotlib
seaborn
```

---
## 🗂️ 데이터 출처

### 유동인구
| 데이터 | 출처 |
|--------|------|
| 버스 노선별·정류장별·시간대별 승하차 인원 | [서울 열린데이터광장](https://data.seoul.go.kr/dataList/OA-12913/S/1/datasetView.do) |
| 정류장별 환승 승객수 및 환승시간 | [서울 열린데이터광장](https://data.seoul.go.kr/dataList/OA-21221/F/1/datasetView.do) |
| 서울시 스마트쉼터 현황 | [서울 열린데이터광장](https://data.seoul.go.kr/dataList/OA-22311/F/1/datasetView.do) |

### 기후 노출 (Exposure)
| 데이터 | 출처 |
|--------|------|
| 열대야 일수 (ERA5-Land) | Copernicus CDS (`cdsapi`) |
| 지표면 온도 (Landsat 8) | Python `geemap` 라이브러리 |
| 도심 미기온 (S-DoT 센서) | [공공데이터포털](https://www.data.go.kr/data/15061244/fileData.do) |
| 자외선 지수 | [기상자료개방포털](https://data.kma.go.kr) |

### 사회취약성 (Sensitivity)
| 데이터 | 출처 |
|--------|------|
| 65세 이상 인구비율, 독거노인, 장애인 등 | [서울 열린데이터광장](https://data.seoul.go.kr) |
| 주택 노후도 | [스마트치안 빅데이터 플랫폼](https://www.bigdata-policing.kr) |

### 적응능력 (Adaptive Capacity)
| 데이터 | 출처 |
|--------|------|
| 무더위 쉼터 현황 | [서울 열린데이터광장](https://data.seoul.go.kr/dataList/OA-21065/A/1/datasetView.do) |
| 서울시 녹지현황 | [서울 열린데이터광장](https://data.seoul.go.kr/dataList/368/S/2/datasetView.do) |


## 📚 참고문헌

- 강수와 외. (2024). 폭염 취약성과 노인 인구를 고려한 무더위 쉼터 입지 연구. *Journal of Climate Change Research*, 15(6), 1001–1022.
- 이만호 외. (2019). 정류소별 영향권 및 접근거리를 반영한 버스 통행 배정 신뢰성 향상 방안 연구. *서울도시연구*, 20(3), 79–90.
- 민진규 외. (2022). 녹지 조성 시나리오에 따른 도시 열환경 개선 효과 분석. *Journal of the Korean Institute of Landscape Architecture*, 50(6), 1–14.
- Kang, Y., Park, J., & Jang, D.-H. (2024). Compound impact of heatwaves on vulnerable groups. *Scientific Reports*, 14(1), 24732.
- Min, J. et al. (2021). Association between income levels and prevalence of heat-related illnesses. *BMC Public Health*, 21(1), 1264.

---
