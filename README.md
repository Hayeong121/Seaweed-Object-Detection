# 김(Seaweed) 이미지 이물질 탐지 AI 모델 개발
> 본 연구는 김 이미지를 활용하여 **이물질을 탐지**하는 AI 모형 개발 프로젝트입니다.
<br>

## 프로젝트 소개
**프로젝트 목적** : 이미지 기반 이물질 탐지 모델을 개발하여, 김 선별 과정에서 이물질을 정확히 식별하여 선별 정확도를 향상시키고 시간 및 비용 절감

**프로젝트 기간** : 2024년 08월 25일 ~ 2024년 10월 28일 (2M)

**역할(구성원)** : 팀장 (총 팀원 4명)

**협업 툴** :  [Notion](https://www.notion.so/Seaweed-AI-1304e0ecc4d180bc8b8ac058a9ab4c9f) <img src="https://github.com/Hayeong121/assets/blob/main/images/Notion-logo.png" width="20">
  
**개발 툴**
* **언어** : Python <img src="https://github.com/user-attachments/assets/3159747f-08b8-423a-bae3-776e5e233be1" width="25">

* **개발환경** : Google Colab <img src="https://github.com/user-attachments/assets/d301ccf8-c112-4567-9fd2-31a32a6b0641" width="40" height="25">
<br>

## 프로젝트 수행 과정

### 1. **데이터 정의**
* 학습용, 검증용, 테스트용으로 분할되어 있으며, 각 분할은 무결함 이미지와 결함이 포함된 이미지로 구성
  <img src="https://github.com/Hayeong121/assets/blob/main/Seaweed-Object-Detection/%EB%8D%B0%EC%9D%B4%ED%84%B0_%EC%A0%95%EC%9D%98_1.png" width="700">
* 김 이미지 내에서 발생할 수 있는 결함은 크게 세 가지 주요 카테고리로 분류
* 각 결함 유형은 다음과 같이 정의<br>
  <img src="https://github.com/Hayeong121/assets/blob/main/Seaweed-Object-Detection/%EB%8D%B0%EC%9D%B4%ED%84%B0_%EC%A0%95%EC%9D%98_2.png" width="700">
     <br>
     <br>
     
### 2. **EDA**
* 결함 유형별 이미지 개수 및 비율 차이가 존재
* fl 유형은 st 유형의 약 절반 수준으로 차이가 상당함 -> 데이터 증강 필요<br>
  <img src="https://github.com/Hayeong121/assets/blob/main/Seaweed-Object-Detection/EDA_1.png" width="700">
* 각 결함 유형별 이미지 내 위치 분포도
* 결함이 상하좌우 약 50 픽셀을 제외한 안쪽에 고르게 분포<br>
  <img src="https://github.com/Hayeong121/assets/blob/main/Seaweed-Object-Detection/EDA_2.png" width="700">
     <br>
     <br>
     
### 3. **데이터 전처리**
#### 3-1. **라벨링 세분화 (클러스터링)**
* 각 유형 내에서도 다양한 형태의 결함이 존재
* 결함의 색상을 이용한 클러스터링을 진행
* 각 유형별 색상에 따른 클러스터의 엘보우 방법 활용 <br>
  <img src="https://github.com/Hayeong121/assets/blob/main/Seaweed-Object-Detection/%EB%8D%B0%EC%9D%B4%ED%84%B0_%EC%A0%84%EC%B2%98%EB%A6%AC_1.png" width="700">
     <br>
* 클러스터링으로 1차 분류 후, 잘못 분류된 라벨은 수동으로 2차 분류 진행
* 최종적으로 다음과 같이 유형 세분화
  * aqua <br>
  <img src="https://github.com/Hayeong121/assets/blob/main/Seaweed-Object-Detection/%EB%8D%B0%EC%9D%B4%ED%84%B0_%EC%A0%84%EC%B2%98%EB%A6%AC_2.png" width="700"><br>
  * floating <br>
  <img src="https://github.com/Hayeong121/assets/blob/main/Seaweed-Object-Detection/%EB%8D%B0%EC%9D%B4%ED%84%B0_%EC%A0%84%EC%B2%98%EB%A6%AC_3.png" width="700"><br>
  * string <br>
  <img src="https://github.com/Hayeong121/assets/blob/main/Seaweed-Object-Detection/%EB%8D%B0%EC%9D%B4%ED%84%B0_%EC%A0%84%EC%B2%98%EB%A6%AC_4.png" width="700"><br>
  
#### 3-2. **데이터 증강**
* 결함 유형 중 floating 유형 이미지의 데이터 불균형을 해소하기 위해 수직/수평 뒤집기 기법을 적용하여 floating 개수 2배 증강<br>
  <img src="https://github.com/Hayeong121/assets/blob/main/Seaweed-Object-Detection/%EB%8D%B0%EC%9D%B4%ED%84%B0_%EC%A0%84%EC%B2%98%EB%A6%AC_5.png" width="700">
* 결함 없는 이미지의 데이터 불균형을 해소하기 위해 Mixup 기법을 적용하여 2600개 증강<br>
  <img src="https://github.com/Hayeong121/assets/blob/main/Seaweed-Object-Detection/%EB%8D%B0%EC%9D%B4%ED%84%B0_%EC%A0%84%EC%B2%98%EB%A6%AC_6.png" width="700">
     <br>
     <br>
     
### 4. **모델링/하이퍼파라미터튜닝**
* YOLO 모델은 입력받은 이미지를 격자(grid) 형태로 나누어 각 셀에서 해당 영역에 객체(결함)가 존재하는지, 그 위치와 종류를 예측함
* YOLOv8n 버전과 YOLOv8s 버전, YOLOv11n 버전, YOLOv11s 버전으로 모델 학습
* 앙상블 기법을 사용하여 결함 검출 성능을 개선<br>
  1. 이진 분류 모델
      * 이미지 내 결함 유무를 빠르게 판단하여, 결함이 없는 이미지에서는 YOLO 모델이 작동하지 않도록 설정
  2. YOLO 모델
      * 결함이 있는 이미지에 대해서만 YOLO가 실행되도록 하여, 결함의 위치와 종류를 세밀하게 검출<br>
    <br>
    <br>
     
### 5. **오류 분석**
* 생성된 예측 데이터와 실제 Validation과 비교하며 모델의 한계점 확인
* 오류를 3가지 경우로 분류 <br>
  1. 실제로 결함이 있으나 예측하는 과정에서 결함을 탐지 못한 이미지 데이터
      * 52개 이미지 데이터에서 오류 발견 (결함률 1.5%)
      * fl 유형에서 25개로 가장 많은 오류, st 유형에서 15개, aq 유형에서 12개의 오류 확인
  2. 실제로는 결함이 없었으나 예측하는 과정에서 결함을 탐지한 이미지 데이터
      * 43개의 이미지 데이터에서 오류 발경 (결함률 1.1%)
      * 육안으로 식별이 불가능한 결함이었던, fl-7 결함 유형에서 가장 많은 오류 확인
  3. 실제로 결함이 있었고 예측하는 과정에서도 결함을 탐지하였으나 정답과 다른 위치에서 결함을 탐지한 이미지 데이터
      * 23개의 이미지 데이터에서 오류 발경 (결함률 0.6%)
      * 모델에서 각 결함 유형의 정확한 특징을 완벽하게 추출하여 학습하는 과정이 미흡함 
     <br>
     <br>

### 6. **결론**
* 본 프로젝트에서는 YOLO v11을 활용하여 김 이미지에서 이물질을 검출하고 품질을 분류하는 모
 개발
* 각 결함 유형을 보다 세부적으로 파악하기 위해 색상 클러스터링을 수행
* 데이터 불균형 해소를 위한 데이터 증강 기법 활용
* 결함 유무를 판별하는 1차 YOLO 기반 이진 분류 모델과 각 결함을 세밀하게 검출하는 2차 YOLO 기반 결함 검출 모델로 구성
* 이진 분류 모델은 Precision 0.95, Recall 0.93을 기록, 결함 검출 모델은 IoU 75% 기준에서 Precision 0.96, Recall 0.89의 우수한 결과 도출.
* 최종 모델에서 특정 결함 유형(ex.fl-7)에서의 학습이 잘 되지 않음. 구분이 어려운 결함에 있어서, 모델의 성능을 높일 수 있도록, 추후 학습을 진행하여 보완 필요
     <br>
     <br>
