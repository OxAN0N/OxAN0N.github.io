---
title : "사이킷런 주요 모듈"
excerpt : "사이킷런 주요 모듈 정리"
categories :
  - ML
tags:
  - ML
  - Python
  - sklearn
last_modified_at : 2023-01-10
---

<table style="border: 2px;">
  <tr>
    <td> 분류 </td>
    <td> 모듈명 </td>
    <td> 설명 </td>
  </tr><tr>
    <td width="130"> 예제 데이터 </td>
    <td> sklearn.datasets </td>
    <td> 사이킷런에 내장되어 예제로 제공하는 데이터 세트</td>
  </tr><tr>
    <td rowspan="4"> feature 처리 </td>
  </tr><tr>
    <td>sklearn.preprocessing</td>
    <td>데이터 전처리에 필요한 다양한 가공 기능 제공 (문자열을 숫자형 코드 값으로 인코딩, 정규화, 스케일링 등)</td>
  </tr><tr>
    <td> sklearn.feature_selection </td>
    <td> 알고리즘에 큰 영향을 미치는 feature를 우선순위대로 selection 작업을 수행하는 다양한 기능 제공 </td>
  </tr><tr>
    <td>sklearn.feature_extraction</td>
    <td>텍스트 데이터나 이미지 데이터의 벡터화된 피처를 추출하는데 사용됨. (ex. 텍스트 데이터에서 count vectorizer나 Tf-ldf Vectorizer 등을 생성하는 기능 제공.)<br> 텍스트 데이터의 피처 추출은 sklearn.feature_extraction.text에, 이미지 데이터의 경우 sklearn.feature_extraction.image에 지원 API가 있음.</td>
   </tr><tr>
    <td>feature 처리 & 차원 축소</td>
    <td>sklearn.decomposition </td>
    <td>차원 축소와 관련한 알고리즘을 지원하는 모듈.<br> PCA, NMF, Truncated SVD 등을 통해 차원 축소 기능을 수행할 수 있음 </td>
   </tr><tr>
    <td>데이터 분리, 검증 & 파라미터 튜닝</td>
    <td>sklearn.model_selection</td>
    <td>교차 검증을 위한 학습용/테스트용 분리, 그리드 서치(GridSearch)로 최적 파라미터 추출 등의 API 제공 </td>
   </tr><tr>
    <td>평가</td>
    <td>sklearn.metrics</td>
    <td>분류, 회귀 클러스터링, 페어와이즈(Pairwise)에 대한 다양한 성능 측정 방법 제공<br>Accuracy, Precision, Recall, ROC-AUC, RMSE 등 제공 </td>
   </tr><tr>
     <td rowspan="8">ML 알고리즘</td>
   </tr><tr>
    <td>sklearn.ensemble</td>
    <td>앙상블 알고리즘 제공 <br>랜덤 포레스트,에이다 부스트, 그래디언트 부스팅 등 제공</td>
   </tr><tr>
    <td>sklearn.linear_model</td>
    <td>주로 선형회귀, Ridge, Lasso 및 로지스틱 회귀 등 회귀 관련 알고리즘을 지원, 또한 SGD(Stochastic Gradient Descent) 관련 알고리즘도 제공</td>
   </tr><tr>
    <td>sklearn.naive_bayes</td>
    <td>나이브 베이즈 알고리즘 제공. 가우시안 NB, 다항분포 NB 등</td>
   </tr><tr>
    <td>sklearn.neighbors</td>
    <td>최근접 이웃 알고리즘 제공, K-NN 등 </td>
   </tr><tr>
    <td>sklearn.svm</td>
    <td>서포트 벡터 머신 알고리즘 제공 </td>
   </tr><tr>
    <td>sklearn.tree</td>
    <td>의사 결정 트리 알고리즘 제공</td>
   </tr><tr>
    <td>sklearn.cluster</td>
    <td>비지도 클러스터링 알고리즘 제공 <br>(K-평균, 계층형, DBSCAN 등)</td>
   </tr><tr>
    <td>유틸리티</td>
    <td>sklearn.pipeline</td>
    <td>피처 처리 등의 변환과 ML 알고리즘 학습, 예측 등을 함께 묶어서 실행할 수 있는 유틸리티 제공</td>
   </tr>


</table>

