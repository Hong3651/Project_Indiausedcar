# 인도 중고차 가격 예측 모델링 

## 1. 프로젝트 개요

- **목적**
  - 중고차의 차량 정보(연식, 주행거리, 연료, 엔진 제원 등)를 기반으로 **실제가격(Price)을 예측**하는 회귀 모델을 개발
  - 고객·딜러 입장에서 **합리적인 시세 산정 기준**을 제시하고, 향후 **자동 견적/추천 서비스의 베이스 모델**로 활용 가능한 수준의 성능 확보

- **진행 기간**
  - 2025.10 (포스코 빅데이터 아카데미, 팀 A3)

- **역할**
  - 팀장 / 데이터 전처리 및 피처 엔지니어링 설계
  - 모델링 파이프라인 구축 및 주요 모델(선형회귀, 트리계열) 비교·평가
  - 핵심 설명변수(vital few) 도출 및 시각화, PPT·리포트 구조 설계

---

## 2. 데이터 개요

- **데이터 출처**: 교육용 중고차 시세 데이터셋 (Kaggle 기반 가공본)
- **주요 컬럼**
  - `Price` : 중고차 실제 거래가(타깃)
  - `Year` : 출고 연도
  - `Kilometers_Driven` : 누적 주행거리
  - `Fuel_Type` : 연료 타입 (Petrol, Diesel, CNG 등)
  - `Transmission` : 변속기(Manual / Automatic)
  - `Owner_Type` : 소유자 유형 (First, Second, Third…)
  - `Mileage` : 연비(kmpl)
  - `Engine` : 배기량(CC)
  - `Power` : 출력(bhp)
  - `Seats` : 좌석 수
  - `New_Price` : 신차 기준가격
  - 그 외 차량명, 제조사 등 카테고리형 컬럼

---

## 3. 데이터 전처리 및 피처 엔지니어링

1. **결측·이상치 처리**
   - `Price == 0` 혹은 명백한 오류값 행 제거
   - `Mileage`, `Engine`, `Power`, `New_Price` 등 문자열+단위 혼합 컬럼을
     - 숫자 + 단위 분리  
     - 숫자만 추출 후 `float`/`int`로 변환
   - 극단 이상치는 IQR/도메인 기준으로 검토 후 제거 또는 클리핑

2. **파생 변수 생성**
   - `vehicle_age` : 분석 기준년도 − `Year`  → 차량 연식
   - `km_per_year` : `Kilometers_Driven` / `vehicle_age` → 연간 주행거리
   - `mileage_per_engine` : `Mileage` / `Engine` → 배기량 대비 연비 효율
   - 필요 시 제조사/차급 등 그룹 변수를 추가해 브랜드·세그먼트 효과 반영

3. **인코딩 및 스케일링**
   - `Fuel_Type`, `Transmission`, `Owner_Type`, `Company` 등 범주형 변수:
     - 고유값 개수에 따라 **One-Hot Encoding / Label Encoding** 적용
   - 트리 모델 중심이라 필수 스케일링은 아니지만,
     - 선형 회귀·규제 모델 실험을 위해 `StandardScaler`로 일부 피처 스케일링

4. **학습 데이터 분리**
   - `train_test_split(test_size=0.3, random_state=42)`
   - 평가는 **홀드아웃 검증** + 주요 모델 간 비교

---

## 4. 모델링 및 평가

1. **Baseline 모델**
   - **다중 선형회귀 (Linear Regression)**
   - Target `Price` 및 일부 연속형 변수에 `log1p` 변환 적용한 버전도 실험
   - 지표: `RMSE`, `MAE`, `R²`

2. **트리 기반 모델**
   - **DecisionTreeRegressor**
   - **RandomForestRegressor**
   - **GradientBoostingRegressor**
   - 각 모델에 대해 기본 파라미터 → 중요 파라미터 중심의 **GridSearchCV** 수행
     - 예: `n_estimators`, `max_depth`, `min_samples_split`, `learning_rate` 등

3. **최종 모델 선정**
   - 선형 회귀 대비 트리 계열 모델에서 **비선형 관계와 상호작용을 잘 포착**
   - 그 중 **GradientBoostingRegressor(튜닝 버전)** 이
     - 가장 낮은 RMSE / MAE
     - 가장 높은 R²
     를 보여 **최종 모델**로 선정

4. **성능 예시** (포맷만, 실제 값은 PPT 기준으로 수정)
   - Train R²: `0.xx`
   - Test R²: `0.xx`
   - Test RMSE: `x.xxx`
   - Test MAE: `x.xxx`

---

## 5. 핵심 설명변수(Vital Few) 인사이트

최종 Gradient Boosting 모델의 `feature_importances_` 상위 변수를 분석한 결과,  
중고차 가격에 특히 영향을 많이 미치는 요인은 다음과 같이 정리됨:

1. **`New_Price` (신차 가격)**  
   - 기본적인 가격 레벨을 결정하는 가장 중요한 변수  
   - 동일 조건이라면 신차가가 높은 차량일수록 중고 가격도 높게 형성

2. **`vehicle_age` (차량 연식)**  
   - 연식이 증가할수록 가격이 하락하는 뚜렷한 패턴  
   - 특히 **초기 3~5년 구간에서 가격 하락 폭이 크게 나타남**

3. **`Kilometers_Driven`, `km_per_year` (주행거리·연간 주행거리)**  
   - 누적 주행거리가 많을수록, 연간 주행량이 과도할수록 가격이 할인  
   - 동일 연식이라면 주행거리가 “정상 범위” 안에 있는 차량이 더 높은 시세를 형성

4. **`Power`, `Engine` (엔진 출력·배기량)**  
   - 고성능/대배기량 차량은 기본 가격이 높게 형성되나  
   - 연비·유지비 등과의 trade-off 로, 특정 구간 이후에는 한계효용 감소

5. **`Owner_Type`, `Fuel_Type`, `Transmission`**  
   - 1인 소유 / 디젤·가솔린 인기 조합 / 오토 변속기 등에서 상대적으로 높은 가격
   - 다수 소유 이력 또는 비선호 연료 조합은 가격 디스카운트 요인으로 작용

이러한 인사이트는
- 딜러 입장에서는 **입고·매입 전략 수립**  
- 플랫폼 서비스에서는 **시세 범위 추천/이상 가격 탐지** 로 활용 가능함.

---

## 7. 폴더 구조 및 실행 방법

```bash
.
├── Code - USedCar_Code       # 전체 분석·모델링 코드 (Jupyter Notebook)
├── ppt&reference - PPT보고서.pdf,참고문헌       # 프로젝트 발표 자료
├── 핵심인자 정리 템플릿.xlsx  # 중요 변수 정리 산출물
└── data/
    └── used_cars_raw.csv            # 원천 데이터 (비공개, 예시)
