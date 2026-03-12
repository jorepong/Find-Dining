# 🍽️ Find-Dining (음식점 리뷰 종합 플랫폼)
<img width="530" height="526" alt="스크린샷 2026-03-13 010150" src="https://github.com/user-attachments/assets/f787fde5-a83e-4a51-82b4-726bd681f2a0" />

Find-Dining은 여러 플랫폼에 흩어진 음식점 리뷰를 하나로 모으고, AI를 활용해 '진실된 리뷰'와 '대가성 리뷰'를 판별하여 소비자에게 왜곡 없는 종합 평점(AI Review Score)을 제공하는 웹 서비스입니다.

## 기획 배경 및 문제 정의
현재 배달 앱 및 지도 플랫폼의 리뷰 시스템은 리뷰 이벤트 등을 통한 '대가성 긍정 리뷰'로 인해 심각하게 오염되어 있습니다. 소비자의 65.2%가 리뷰 이벤트 참여를 위해 후기를 작성하며, 이는 실제 만족도보다 높은 별점으로 이어져 리뷰의 본질적인 신뢰도를 훼손합니다.
Find-Dining은 정보의 비대칭성을 해결하고, 조작된 평점을 걸러내어 사용자의 합리적인 소비를 돕고자 기획되었습니다.


## 핵심 기능 및 AI 로직

### 1. 진실 리뷰 vs 대가성 리뷰 분류
텍스트만으로는 대가성 여부를 판단하기 어렵다는 한계를 돌파하기 위해, 통계적 접근과 머신러닝을 결합했습니다.

* 데이터셋 구축 로직: 3개 플랫폼(네이버, 배민, 쿠팡이츠)을 교차 검증하여 리뷰 이벤트를 전혀 하지 않는 식당의 리뷰를 '진실'로, 이벤트 메뉴가 언급된 리뷰를 '대가성'으로 라벨링하여 총 2만 개의 기준 데이터셋을 구축했습니다.
* 통계적 유의미성 확인: 두 집단을 T-Test 분석한 결과, 진실 리뷰에서 명사와 조사가 더 많이 쓰이고 글의 길이가 더 길다는 유의미한 차이를 식별했습니다.
* 모델 적용: 한국어 형태소 분석기(Okt)와 TF-IDF로 벡터화한 뒤, SVM(Support Vector Machine)을 적용하여 77%의 분류 정확도를 달성했습니다.

<img width="5904" height="1948" alt="Mermaid Chart - Create complex, visual diagrams with text -2026-03-12-155455" src="https://github.com/user-attachments/assets/61dccda8-379d-4e0b-9b02-0ac201fa351d" />


### 2. 리뷰 감성 분석 및 할루시네이션 프리 요약

* 감성 분석: 5,500개의 리뷰 데이터를 라벨링하고, SMOTE 기법으로 데이터 불균형을 해소한 뒤 Naive Bayes Classifier를 통해 긍정/부정/중립을 약 90% 정확도로 분류합니다.
* 조건부 요약: LLM의 할루시네이션 현상을 방지하고 API 비용을 절감하기 위해, 감성 분석으로 사전 필터링된 20자 이상의 리뷰들만 그룹화하여 요약을 진행합니다.

### 3. 신뢰 지표: AI Review Score (1~100점)
단순 평균이 아닌, 모델의 분류 결과와 신뢰도를 모두 수학적으로 반영한 자체 스코어링 수식을 설계했습니다.
<img width="5644" height="1360" alt="Mermaid Chart - Create complex, visual diagrams with text -2026-03-12-155739" src="https://github.com/user-attachments/assets/0bae0b0a-0211-44a1-a20d-37bac0e1a6c5" />

* 산출 수식: $AI~Score=0.5\times(R_{scaled}+S)\times(1-P\times A\times a)$ 
* $R_{scaled}$: 기성 플랫폼의 1\~5점 평점을 1\~100점 척도로 정규화한 값.
* $S$: 긍정 및 중립 리뷰의 비중을 가중 계산한 감정적 경향성 점수 $S=\frac{N_{pos}+(0.5N_{neu})+(0N_{neg})}{N_{total}}\times100$.
* 패널티 항목: 대가성 리뷰 비율($P$), 모델 분류 정확도($A$), 조정 계수($a$, 0.7 적용)를 곱하여, 대가성 리뷰가 많을수록 치명적인 패널티를 부여합니다.


## ⚙️ 아키텍처 및 기술 스택
<img width="3951" height="4852" alt="Mermaid Chart - Create complex, visual diagrams with text -2026-03-12-155831" src="https://github.com/user-attachments/assets/14a72ebd-ac11-4655-9e30-63a2ffe9a8ea" />

### Backend & AI (Core)

* Framework: Django REST Framework 
* AI/ML: Python, Scikit-learn (SVM, Naive Bayes), SMOTE, TF-IDF, Okt (형태소 분석기)
* NLP & API: OpenAI API (gpt-4o)
* Crawling: Selenium / BeautifulSoup (네이버 지도, 배달의민족, 쿠팡이츠 데이터 수집)

### Frontend
* Framework: React


## 👥 팀 구성 및 역할
* 이수연: Backend, DB 설계 및 API 개발 
* 조현빈: Backend, 리뷰 크롤러 파이프라인 구축, AI 분류 모델(진실/대가) 설계 및 학습 
* 홍승민: Backend, AI 감성 분석 모델 설계 
* 이민정, 홍준기: Frontend, UI/UX 디자인 및 데이터 시각화 
