# 프로젝트4 : 영상 이미지의 폭력 상황을 감별하는 딥러닝 모델 구현

# 1. Overview

## 1.1. 배경과 목적
- 공공장소, 치안 사각지대에서 발생하는 잦은 폭력문제에 발빠른 조치와 더불어 미연에 방지하는 것이 목적
- CCTV영상, 보안 카메라 영상을 활용하여 폭력 상황을 감별하는 모델 구현

<br/>

## 1.2. 데이터 소개
- 캐글 출처, 유튜브 영상 자료
- 폭력 영상 1,000개 / 비폭력 영상 1,000개 / 러닝타임 3~5초

<br/>

## 1.3. 스킬셋
- numpy
- matplotlib
- seaborn
- openCV
- base64
- imageio
- imgaug
- scikit-learn
- tensorflow
- keras



---
<br/>

# 2. 데이터 전처리
- 학습 시 발생하는 GPU 메모리 문제로 폭력, 비폭력 각각 1000개의 영상 중 370개 사용
- openCV 활용 영상 6프레임마다 이미지를 캡쳐 후 BGR-RGB 변환 및 이미지 사이즈 조정(96)
- 이미지 좌우 반전, 확대, 회전 등 이미지 증강 작업을 통해 중복되거나 비슷한 데이터를 방지하고 학습 능률을 제고함  
 ![image](https://user-images.githubusercontent.com/110115061/223902747-2ba58914-c1bb-43b9-acec-c4865f35c64c.png)  
- 데이터 정규화 작업
- 훈련, 검증, 평가 데이터 크기
  - 훈련 데이터 : (10787, 96, 96, 3)
  - 검증 데이터 : (2697, 96, 96, 3)
  - 평가 데이터 : (3372, 96, 96, 3)



---
<br/>

# 3. 모델링

## 3.1. 모델 소개
- 모델명 : MobileNetV2
- 제한된 전력 및 컴퓨터 환경을 위해 설계된 CNN 구조
- 주요 특징
  - Depthwise Separable Convolution
  - Inverted Residual Block
  - Linear Bottleneck


### 3.1.1. Depthwise Separable Convolution  
![image](https://user-images.githubusercontent.com/110115061/223911812-67d68fa0-4856-447a-a6d9-4a3ff64611c5.png)  
- 각 채널별로 convolution 연산 이후 1*1 convolution 연산으로 깊이 방향 연산 수행
- 기존 CNN모델보다 연산량 감소
- 비선형성 증가로 복잡한 패턴 더 잘 인식함

### 3.1.2. Inverted Residual Block & Linear Bottleneck  
![image](https://user-images.githubusercontent.com/110115061/223914705-85c71802-3afe-406a-a5e2-d3a8cbce8b60.png)  
- 기존 Residual block의 순서와 거꾸로 진행함
- 차원 확장 -> 정보 추출 -> 차원 축소
- Skip connection을 활용한 잔차학습 과정에서 방법 개선
- 메모리 사용 측면에서 효율적

<br/>

## 3.2. 모델 구현 및 학습
- GPU 메모리 이슈 : 주피터노트북 커널 충돌 및 잦은 메모리 관련 에러로 학습에 어려움 발생
- 메모리 부족 문제 해결과 더불어 효율적 학습 및 좋은 성능을 위해 모델 파라미터를 직접 변경하며 여러번 학습 시도하였음

### 3.2.1. 학습
- 이미지 사이즈 : 96
- epochs : 100
- early stopping : patience 5, val_loss
- ReduceLROnPlateau : patience 2  
![image](https://user-images.githubusercontent.com/110115061/223918817-6f4c76ca-e10c-4473-a750-6217d06b1ad3.png)  
- best epochs : 14
- train accuracy : 0.97..
- train loss : 0.08..
- valid accuracy : 0.93..
- valid loss : 0.15..

<br/>

## 3.3. 성능 평가
- 평가 데이터로 모델 평가
- 혼동행렬  
![image](https://user-images.githubusercontent.com/110115061/223919334-e495de3c-3949-4c8c-8f68-aac7ea4a5421.png)  
- classification report
![image](https://user-images.githubusercontent.com/110115061/223919425-4a9122a5-b337-4f71-8305-407589d8107d.png)  
- 정확도와 f1 스코어 모두 높은 값을 보여 꽤 높은 정확도로 폭력상황 여부를 잘 감지하는 것을 확인할 수 있음
- 모델의 영상 판별 시각화  
![image](https://user-images.githubusercontent.com/110115061/223919858-37d1a71c-ed7a-4422-8b78-d90ca70b7f25.png)  



---
<br/>

# 4. 결론
- 구현한 모델이 폭력상황을 잘 감별하는 것을 볼 수 있음
- 적용
  - 실제 CCTV 관제센터에 적용, 폭력상황 감지 시 알람을 울리는 방식
  - 어둡거나 후미진 곳이라면 상황 감지 시 굉장히 밝은 조명을 쏘는 방식
  - 상황 감지 시 인근 파출소 등에 상황 보고가 되는 방식
- 활용점을 넓혀보자면 폭력 이외의 다른 범죄행위에도 적용해볼 수 있을 것



---
<br/>

# 5. 한계 및 개선점
- 실제 CCTV영상 데이터로 모델링과 학습이 진행될 필요가 있음(gray scale, 어두움). 하지만 개인정보보호 문제 해결 필요
- 실제로 적용하기엔 CCTV를 통한 자료 수집, 프레임별 이미지 추출 및 모델 적용 시간이 아직은 충분히 빠르지 않음
- 영상 데이터 전체적으로 폭력과 비폭력 이미지의 차이가 큼
  - 더 많고 다양한 상황의 영상데이터를 학습할 필요가 있음(리소스 문제 해결할 필요 있음)
