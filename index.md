---
layout: default
---

***Based on [One-shot Voice Conversion by Separating Speaker and Content Representations with Instance Normalization](https://github.com/jjery2243542/adaptive_voice_conversion)***

## Introduction

## Dataset

총 3개의 데이터셋을 사용하여 훈련 및 결과 확인

Train과 Test비율은 동일하게 사용

| Dataset      | Detail          | 길이  | 화자수 |
| ------------ | --------------- | --- | --- |
| VCTK Dataset |                 | 1   |     |
| LibriSpeech  | train-clean-100 | 2   | 2   |
| LibriSpeech  | train-clean-360 | 3   | 3   |

## 기존 논문과 코드 비교

원 저자는 기존 논문에서 제안한 방법에 몇가지 수정을 한 코드를 공유하였고, 차이점은 다음과 같습니다.

1. 기존 논문에서의 Lambda값 사용
2. Reconstruction Loss Fuction
3. Normalization의 위치
4. log sigma값 사용 여부

## Demo Page와의 비교

Demo Page에서는 VCTK Dataset을 사용하여 음성을 생성하였습니다.

Github에 Pre-Trained된 모델을 공개하여, 같은 방법으로 전처리 및 훈련을 진행하였고 유사한 결과가 나오는지 체크하였습니다.

## Results

### Decoder Test

source와 Target을 동일하게 설정한 Inference 결과를 통해 Decoder의 성능을 확인할 수 있다.

### Seen / Unseen Test


## Conclusion