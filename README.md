# Recommender-System
구직자 맞춤 채용 공고 추천 시스템 (제1회 국민대학교 AI빅데이터 분석 경진대회)

## 주제
**이력서 맞춤형 채용 공고 추천 AI 모델 개발**

이력서, 채용 공고 및 지원 히스토리 데이터를 활용하여 구직자에게 맞춤화된 채용 공고를 자동으로 추천할 수 있는 추천시스템 알고리즘 개발 

## 평가 지표
Recall@5

## 목표
구직자의 이력서 정보와 채용 공고 정보를 비교하여 적합한 공고를 추천

TF-IDF 벡터화 및 코사인 유사도를 활용한 추천 시스템 구현 (콘텐츠 기반 필터링) 및
이 프로젝트는 사용자(구직자)와 채용 공고 간의 최적의 추천을 생성하기 위해 여러 가지 추천 기법을 결합하는 앙상블 기반 추천 시스템을 구축하는 것을 목표로 한다. (협업 필터링)

## 협업 필터링 기반

사용자(구직자)와 아이템(채용 공고)간의 추천을 수행하는 여러 가지 유사도 방법을 결합하여 최적의 추천을 생성하는 구조

### **1. 데이터 전처리**

**(1) 구직자-채용 공고 매칭 데이터 변환**
- 구직자(resume_seq)와 채용 공고(recruitment_seq)간의 rating_total 데이터를 피벗 테이블(pivot table) 형태로 변환
- 결측값(NA)은 0으로 채워 공고 지원 여부를 명확하게 표현

**(2) 학습(train) / 검증(val) 데이터 분리**
- 각 구직자가 지원한 채용 공고 리스트를 생성
- 리스트를 shuffle 한 뒤, 마지막 공고를 검증 세트(val)로 분리하고 나머지는 학습 세트(train)로 사용.

### **2. 사용자-아이템 행렬 기반 추천 시스템 구축**
   
**(1) SVD(특이값 분해)를 이용한 잠재 요인(latent factor) 추출**
- 사용자-아이템 행렬을 기반으로 SVD 수행
- 차원을 줄여 잠재 요인(latent factor)을 추출, 주요 특성 k=50 개만 유지하여 행렬을 근사
- 예측된 사용자-아이템 행렬을 저장: train_user_predicted_ratings_svd

**(2) 코사인 유사도를 활용한 추천 점수 생성**
- 채용 공고 간의 코사인 유사도(Cosine Similarity) 계산
- 유사도를 활용하여 사용자별 추천 점수를 생성
- 예측된 사용자-아이템 행렬을 저장: train_user_predicted_scores_cosine

**(3) 피어슨 상관계수를 활용한 추천 점수 생성**
- 사용자 간의 피어슨 상관계수를 계산하여 유사도 측정
- 유사도를 기반으로 추천 점수를 계산
- 예측된 사용자-아이템 행렬을 저장: train_user_predicted_scores_pearson

**3. 최종 추천 생성 - 앙상블(Ensemble)**
- 세 가지 추천 점수(SVD, 코사인 유사도, 피어슨 상관계수)를 결합하여 최적의 추천을 생성
- 추천 점수 앙상블 방법:
   - 각 방법에서 예측된 점수를 조합하여 가중 평균
   - 추천된 공고 중 가장 많이 등장한 상위 5개를 최종 추천

**검증, 성능 평가 - (Recall@5)**
- recall@5 지표를 활용하여 추천된 채용 공고가 실제 검증 데이터(val)와 얼마나 일치하는지 평가

## 콘텐츠 기반 필터링 (시도)

### 1. 이력서 데이터 전처리
   
**(1) resume_certificate.csv - 자격증 데이터 전처리**
분석: 
- Certificate_contents(자격증 내용) 열에는 동일한 의미를 갖는 여러 표현이 포함됨.
   - 예시: '자동차운전면허 2종 보통', '운전면허2종보통', '2종운전면허증' → 동일한 자격증을 의미

해결:
- 특정 키워드 기반 전처리 적용
   - 예: '2종', '1종', 'MCAS MASTER' 등 주요 키워드만 남기도록 변환

**(2) resume.csv - 직무 키워드 데이터 전처리**
분석:
- text_keyword(직무 키워드 및 채용 공고 키워드) 열의 단어들이 ;(세미콜론)으로 구분되어 있음.
- TF-IDF 벡터화 시 개별 단어(토큰) 단위로 가중치를 계산하므로 ; 같은 구분자는 제거해야 함.

해결:
- ; 구분자를 공백(' ')으로 대체하여 단어 분리 효과를 유지하면서 TF-IDF 벡터화가 가능하도록 변환

### 2. 콘텐츠 기반 필터링 기반 추천 시스템 구축

**(1) TF-IDF 벡터화**
- text_keyword(직무 키워드 및 채용 공고 키워드) 데이터를 TF-IDF 벡터화하여 수치 데이터로 변환

**(2) 코사인 유사도(Cosine Similarity) 계산**
- 구직자 이력서 벡터와 채용 공고 키워드 벡터 간의 코사인 유사도를 계산
- 구직자가 특정 채용 공고에 얼마나 적합한지 측정

**(3) TF-IDF 기반 행렬 생성**
- TF-IDF 기반 코사인 유사도 행렬을 생성
- 행렬의 각 값은 해당 구직자가 특정 채용 공고에 대해 적합도 점수를 가지도록 구성
