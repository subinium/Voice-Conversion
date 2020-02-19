---
layout: default
---

***Based on [One-shot Voice Conversion by Separating Speaker and Content Representations with Instance Normalization](https://github.com/jjery2243542/adaptive_voice_conversion)***

## I. Introduction

<figure>
    <img src="https://github.com/jjery2243542/adaptive_voice_conversion/raw/public/model.png" width="60%" alt>
    <figcaption>Simple Model</figcaption>
</figure>

### I-1. 논문의 아이디어

Utterance에 담긴 정보를 2개의 Encoder를 사용하여 Speaker와 Content 정보로 나누고, Decoder를 사용하여 합치는 Style Transfer 방법을 제시하고 있습니다.



### I-2. 모델 설명


<img src="./model2.png" width="40%" style="float:left; margin: 0px 40px 10px 0px; border : 1px solid gray;" alt>
    

모델은 총 3개의 파트로 구성이 되어 있습니다.

1. **Speaker Encoder** : 화자 정보를 추출합니다.

2. **Content Encoder** : Text 정보를 추출합니다. (Instance Normalization(IN)을 사용하여 Content 정보를 유지하며 Speaker 정보를 제거할 수 있게 만든다고 가정 (TSNE를 활용하여 Clustering을 확인할 수 있지만 미진행))

3. **Decoder** : Content Encoder에서 받은 latent vector를 사용하여 Reconstruction합니다. 이 과정에서 speaker 정보를 Affine transform하여 AdaIN layer에 붙여서 사용합니다.

<br style="clear:both" />

## II. Dataset

총 3개의 데이터셋을 사용하여 훈련하였습니다.

| Dataset     | Detail          | Hours | 평균 길이(sec) | 화자수 | 여성 화자수 | 남성 화자수 | Format | SR    |
| ----------- | --------------- | ----- | ---------- | --- | ------ | ------ | ------ | ----- |
| VCTK Corpus |                 | 44    | 3          | 109 |        |        | .wav   | 24kHz |
| LibriSpeech | train-clean-100 | 100.6 | 12         | 251 | 125    | 126    | .flac  | 16kHz |
| LibriSpeech | train-clean-360 | 363.6 | 12         | 951 | 439    | 482    | .flac  | 16kHz |


## III. Preprocessing & Training

### III-1. Preprocessing

각 음원에 대한 전처리는 다음과 같이 진행하였습니다.

1. Trimmed out the silence
2. Normalize the value
3. Convert the audio to 24kHz
4. STFT with a 50ms window length, 12.5ms hop length, 2048 STFT window size
5. 512-bin Mel-scale spectrograms

전체적인 전처리 과정은 다음과 같습니다.

- 음성 데이터 전체 리스트 random shuffle
- train, val, test set으로 분할 (train : val = 19 : 1)
- 각 리스트의 음원을 mel-spectrogram으로 변환 (박규봉님의 tacotron 내부 코드 사용)
- normalization을 위해 일정 길이의 데이터에서 평균과 분산값 저장 
- mel-spectrogram에서 segment size가 128 이하인 음원은 제거

### III-2. Training

각 데이터셋에 대하여 train은 약 500,000 iters로 훈련을 돌렸고, 각 Loss값은 다음과 같이 줄어드는 것을 볼 수 있습니다.

## IV. 기존 논문과 코드 비교

원 저자는 기존 논문에서 제안한 방법에 몇가지 수정을 한 코드를 공유하였고, 차이점은 다음과 같습니다.

1. **기존 논문에서의 Lambda값 사용** : iter수에 따라 annealing하여 사용
2. **Reconstruction Loss Fuction** : 기존 VAE에서 사용하는 (Gaussian) KLD Loss 사용
3. **log sigma값 사용 여부** : Inference 시, 분산값 사용안함 (사용해본 결과 악화)
4. 그 외 : Dropout 사용안함

그 외의 dummy code 등이 있었으며, 일부 제거하였습니다.

## V. Results

- 생성된 Mel-Spectrogram은 Griffin Lim을 사용하여 wav 파일로 변환하였습니다.
- seen2seen은 Source의 데이터는 본 목소리 중에 train에 실제 사용된 데이터, Target은 변환 점검을 위해 본 목소리지만 train에 사용되지 않은 데이터 입니다.

### V-1. VCTK & Demo 비교

Demo Page에서는 VCTK Dataset을 사용하여 음성을 생성하였습니다.
Github에 Pre-Trained된 모델을 공개하여, 같은 방법으로 전처리 및 훈련을 진행하였고 유사한 결과가 나오는지 체크하였습니다.

- 기존 [Demo](https://jjery2243542.github.io/one-shot-vc-demo/)
- Male(seen)에서 변환된 목소리에서 Content(Text) 정보가 사라진 것을 볼 수 있습니다.


<table>
    <thead>
        <th></th>
        <th>wav</th>
        <th>Male(seen) p246-343</th>
        <th>Female(seen) p299-241</th>
    </thead>
    <tbody>
        <tr>
            <th>wav</th>
            <td></td>
            <td> 
                <audio controls> <source src='./assets/p246-343.wav' type='audio/x-wav'> </audio>
            </td>
            <td>
                <audio controls> <source src='./assets/p299-241.wav' type='audio/x-wav'> </audio>
            </td>
        </tr>
        <tr>
            <th>Male(Seen) p292-052</th>
            <td> 
                <audio controls> <source src='./assets/p292-052.wav' type='audio/x-wav'> </audio>
            </td>
            <td>
                <audio controls> <source src='./assets/vctk-s2s-m2m.wav' type='audio/x-wav'> </audio>
            </td>
            <td>
                 <audio controls> <source src='./assets/vctk-s2s-f2m.wav' type='audio/x-wav'> </audio>
            </td>
        </tr>
        <tr>
            <th>Female(Seen)p225-124</th>
            <td>
                <audio controls> <source src='./assets/p225-124.wav' type='audio/x-wav'> </audio>
            </td>
            <td>
                <audio controls> <source src='./assets/vctk-s2s-m2f.wav' type='audio/x-wav'> </audio>
            </td>
            <td>
                <audio controls> <source src='./assets/vctk-s2s-f2f.wav' type='audio/x-wav'> </audio>
            </td>
        </tr>
    </tbody>
</table>


### V-2. train-clean-100

- 데이터가 VCTK에 비해 잡음이 많음 -> 잡음에 따라 convert된 음질 차이가 큼

<table>
    <thead>
        <th></th>
        <th>wav</th>
        <th>Male(seen) </th>
        <th>Female(seen) </th>
    </thead>
    <tbody>
        <tr>
            <th>wav</th>
            <td></td>
            <td> 
                <audio controls> <source src='./assets/8629-261140-0017.flac'> </audio>
            </td>
            <td>
                <audio controls> <source src='./assets/6147-34605-0021.flac'> </audio>
            </td>
        </tr>
        <tr>
            <th>Male(Seen) </th>
            <td> 
                <audio controls> <source src='./assets/3214-167607-0029.flac'> </audio>
            </td>
            <td>
                <audio controls> <source src='https://raw.githubusercontent.com/subinium/Voice-Conversion/master/assets/libri100-s2s-m2m.wav' type='audio/wav'> </audio>
            </td>
            <td>
                 <audio controls> <source src='https://raw.githubusercontent.com/subinium/Voice-Conversion/master/assets/libri100-s2s-f2m.wav' type='audio/wav'> </audio>
            </td>
        </tr>
        <tr>
            <th>Female(Seen)</th>
            <td>
                <audio preload="auto" controls> <source src='./assets/3879-174923-0002.flac'> </audio>
            </td>
            <td>
                <audio preload="auto" controls> <source src='https://raw.githubusercontent.com/subinium/Voice-Conversion/master/assets/libri100-s2s-m2f.wav' type='audio/x-wav'> </audio>
            </td>
            <td>
                <audio  preload="auto" controls> <source src='https://raw.githubusercontent.com/subinium/Voice-Conversion/master/assets/libri100-s2s-f2f.wav' type='audio/x-wav'> </audio>
            </td>
        </tr>
    </tbody>
</table>

### V-3. train-clean-360

<table>
    <thead>
        <th></th>
        <th>wav</th>
        <th>Male(seen) </th>
        <th>Female(seen) </th>
    </thead>
    <tbody>
        <tr>
            <th>wav</th>
            <td></td>
            <td> 
                <audio controls> <source src='./assets/5672-75791-0006.flac'> </audio>
            </td>
            <td>
                <audio controls> <source src='./assets/9023-296467-0001.flac'> </audio>
            </td>
        </tr>
        <tr>
            <th>Male(Seen) </th>
            <td> 
                <audio controls> <source src='./assets/7540-101799-0002.flac'> </audio>
            </td>
            <td>
                <audio preload="auto" controls> <source src='https://raw.githubusercontent.com/subinium/Voice-Conversion/master/assets/libri360-s2s-m2m.wav' type='audio/x-wav'> </audio>
            </td>
            <td>
                 <audio  preload="auto" controls> <source src='https://raw.githubusercontent.com/subinium/Voice-Conversion/master/assets/libri360-s2s-f2m.wav' type='audio/x-wav'> </audio>
            </td>
        </tr>
        <tr>
            <th>Female(Seen)</th>
            <td>
                <audio  preload="auto" controls> <source src='./assets/4807-26852-0028.flac'> </audio>
            </td>
            <td>
                <audio  preload="auto" controls> <source src='https://raw.githubusercontent.com/subinium/Voice-Conversion/master/assets/libri360-s2s-m2f.wav' type='audio/x-wav'> </audio>
            </td>
            <td>
                <audio preload="auto" controls> <source src='https://raw.githubusercontent.com/subinium/Voice-Conversion/master/assets/libri360-s2s-f2f.wav' type='audio/x-wav'> </audio>
            </td>
        </tr>
    </tbody>
</table>

## Conclusion

Mel-Spectrogram에서 보이는 것처럼 높은 주파수의 음역대에서 파형이 모두 흐릿해지는 것을 관찰할 수 있습니다.
기존 논문의 결과와 $$A\rightarrow A$$ 변환 등 높은 주파수 음역대에서는 비슷하게 흐려지는 결과를 얻을 수 있습니다.





또한 duration 정보는 그대로 유지하며 Mel-Spectrogram이 생성됩니다.



## Future Work

- dropout과 log sigma 값 등의 hyperparameter 조정을 할 수 있음
  - 저자가 이미 testing 해서 나온 결론으로 보여짐
- Content Encoder 대신에 Text로 전달하여 STS가 아닌 TTS로 구성
