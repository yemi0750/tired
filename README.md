# 피곤하쥬?

## 기간
19.10.30 ~ 19.12.19

## Discription

‘피곤하쥬?’ 시스템은 강의실 내 강의자의 수업 내용을 녹음 및 분석하고, 이를 텍스트화하여 서버에 저장한다. <br>
동시에 수업을 듣는 학생들의 심장박동을 센서로 측정하여 일정 심박이하로 뛸 경우 졸고 있다고 판단, 수면 시간 정보를 저장한다. <br> 
녹음된 음성은 특정 키워드, 혹은 일정 데시벨 이상인 부분은 하이라이팅하여 텍스트화 한 뒤 저장한다. <br>

## Architecture Diagram

![image](https://user-images.githubusercontent.com/38336997/71110210-30aec980-220a-11ea-8d43-b7ee7528c633.png)

## Architecture Discription

* 강의자료 STT(speech to text) 모듈
1. 강의자가 녹음을 시작할 경우 마이크를 통해 라즈베리파이에 녹음 파일이
저장된다. 동시에 클라이언트를 통해 실행된 실행프로그램은 웹캠을 통해
실시간으로 눈의 세로 간격으로 졸음을 파악하여 수면구간을 APIgateway로
전송한다.
2. AWS API Gateway와 lambda를 통해 key, email, sleeptime(String) 정보를 받고
수면시간을 분할하여 list 형태로 변환하여 DB에 저장한다.
3. 녹음이 종료되면 라즈베리파이에 녹음된 Raw data(음성)를 S3에 업로드한다.
4. Raw data가 완전히 업로드 된 후, 각 강의에 대한 모든 수강생의 수면시간을
병합하고, 수면시간에 따라 Raw data를 분할하여 preprocessed data(음성)를 생성해
S3에 업로드한다.
5. Transcribe가 preprocessed data를 input data로 사용하여 json파일을 S3에
업로드한다.
6. S3의 json파일을 가공해 가독성을 높이고, 빈출 단어를 추출해 html 형식으로
만든다.
7. AWS SES 서비스를 이용하여 사용자에게 정리된 html형식을 email로 전송한다.
강의 음성 분석 모듈
1. 강의 음성을 1분단위로 tokenizing 하고 나머지는 절삭한다.
2. 이를 mfcc 특징벡터로 변환하는데, 전처리과정, 프리-엠퍼시스, 윈도윙 과정은
제거하였는데 이는 소리의 파형에서 사람 목소리를 제외한 잡음과 불필요한 무음을
제거하는데, 본 분석에서는 잡음, 강의자의 빠르기 등도 중요한 특징이므로 이
과정을 생략한다.
3. Mfcc로 변환 과정 중 과다한 연산과 음정보정을 위해 window size를 100ms로
지정한다.
4. 기존에 STT모듈을 사용하며 적재된 수면 데이터와 강의자의 mp3 파일을 모두
mfcc 변환을 통해 dtw 알고리즘으로 가장 유사한 음성을 찾아낸다.
5. 그 음성과의 Correlation을 구하여 강의의 전반적인 음성이 기존 수면데이터와의
유사도의 척도를 그려주는 그래프를 그려준다.

## Abstract

‘피곤하쥬?’ 시스템은 강의실 내 강의자의 수업 내용을 녹음하고, 동시에 수업을 듣는 학생들의 영상을 촬영한다. 수업 중 학생들의 영상에서 눈 부분을 인식하여 평균 눈의 세로 길이가 급격하게 줄어들 경우 졸고 있다고 판단, 분 단위로 수면 시간을 계산하여 해당 정보를 전송한다. 녹음된 음성 파일은 S3에 완전히 업로드 되면, 각 강의에 따라 학생들의 수면시간 정보 병합을 실행한다. 업로드된 음성파일은 병합된 수면시간에 따라 분할되어 S3에 다시 저장되고, 이를 transcribe로 전달하여 텍스트화한다. 텍스트화 된 음성파일은 텍스트 가공 모듈을 통해 보기 좋은 형태로 정리되고, 자주 나온 단어를 등장 횟수에 따라 내림차순으로 정렬해 html 형식으로 사용자의 email로 전달된다.
강의자의 음성이 얼마나 졸린 목소리인지 측정하는 모듈은 강의자의 음성을 분 단위로 나누어 mfcc 특징벡터로 변환한다. 기존의 적재된 수면 시 녹음 된 데이터도 mfcc 특징벡터로 변환하여 dtw 알고리즘을 통하여 유사도를 검사한다. 가장 유사한 음성파일과 Correlation을 구해 자신의 강의가 얼마나 수면음성과 비슷한지 그래프로 시각화 한다.

## Result


![image](https://user-images.githubusercontent.com/38336997/71110419-956a2400-220a-11ea-9615-4f0410393752.png)
![image](https://user-images.githubusercontent.com/38336997/71110489-bdf21e00-220a-11ea-9f2b-d63a46a2c0e9.png)
