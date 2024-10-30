# [빌보드 Weekly Hot100 데이터를 통한 음악 트렌드 분석 및 음원 전략 제안]

## 목차
[1. 프로젝트 개요](#1-프로젝트-개요)<br>
[2. 문제 정의](#2-문제-정의)<br>
[3. 데이터 수집](#3-데이터-수집)<br>
[4. 데이터 전처리](#4-데이터-전처리)<br>
[5. 데이터 분석](#5-데이터-분석)<br>
[6. 모델링](#6-모델링)<br>
[7. 결론](#7-결론)<br>
[8. 보완점 발전 방향](#8-보완점-발전-방향)<br>

---
## 1. 프로젝트 개요
* 기간: 2024.10.16 ~ 2024.10.30
* 사용 언어: Python
* 주요 패키지: selenium, BeautifulSoup, pandas, numpy, nltk, colorthief ...  

## 2. 문제 정의
* 목표: 빠르게 변화하는 글로벌 음원 트렌드에 Data-Driven한 비즈니스 의사결정 방식 제안
* 기존 방법<br>
    A&R 방식) 주로 경험과 직관, 그리고 음악 전문가들의 창의적 판단에 의존
* 제안 방향<br>
    데이터 기반 의사결정: 데이터 분석과 통계적 인사이트에 기반한 의사결정 방식
* 개선 방법<br>
    다차원의 음원 관련 데이터 구축 및 인사이트 도출, 빌보드 차트 기준 Top20를 Target으로 한 분류 모델 구축
* 기대 효과
    * 시장 적응력 상승: 글로벌 음원 시장의 트렌드를 파악하여 그에 맞춘 전략 수립
    * 리스크 감소: 데이터에 기반한 예측으로 직관에 의존하는 것보다 높은 정확도 기대

## 3. 데이터 수집
### Billboard HOT100
* 출처: [Billboard HOT100](https://www.billboard.com/charts/hot-100/)
* 수집 구분: Crawling
* 수집 방법: BeautifulSoup4
* 범위: 2014 ~ 2023년의 Weekly 차트 
* 데이터 사이즈: (52100, 6)
* 데이터 구성 <br>
    > Year: 년도 <br> 
    > Month: 월 <br>
    > Week: 주 <br>
    > Rank: 순위 <br>
    > Title: 곡 제목 <br>
    > Artist: 가수 <br>
### 음원 데이터 
#### Song Info
* 출처: [Spotify API](https://developer.spotify.com/documentation/web-api)
* 수집 항목: 곡의 길이(duration_sec), 앨범 커버 링크(album cover), BPM
* 수집 구분: API
* 범위: 2014 ~ 2023 billboard HOT100에 오른 곡들
* 데이터 사이즈: (52100, 3)
#### Lyrics
* 출처<br>
    * 1차) [lyrics/ovh](https://lyricsovh.docs.apiary.io/#)
    * 2차) [Genie](https://www.genie.co.kr/)
    * 3차) [Google](https://www.google.com/)
* 수집 항목: 가사 
* 수집 구분: API, Crawling
* 수집 방법: Selenium, BeautifulSoup4
* 범위: 2014 ~ 2023 billboard HOT100에 오른 곡들
* 데이터 사이즈: (52100, 1)
## 4. 데이터 가공
### 데이터 전처리
#### Genre
    1. 복수 장르 시, 대표값 1개만 지정( ',' 이후 값 삭제)
    2. 부분 중복 데이터 하나로 병합 (예: 랩/힙합 + 힙합 -> 랩/힙합)
#### Duration
    단위: ms -> sec 
#### Featuring
    1. Title의 파생 변수
    2. 구분: 0(Non-Featuring), 1(Featuring)
#### BPM
    1. 소수점 3자리까지만 표기
    2. 형식: 연-월 통합 (Year + Month => Year-Month)
#### Lyrics
    1. 공백 행 제거
    2. 가사 안내(chorus, pre-chorus, intro, outro 등) 제거
    3. 감탄사('yeah', 'woah', 'oh', 'uh', 'ah', 'na', 'la' 등) 제거 
### 데이터 가공
#### Lyrics
##### 텍스트 처리
>1. <b>Tokenization: 단어 토큰화</b>
>2. <b>Cleaning: 노이즈/불용어 제거 - 영어, 스페인어</b>
>3. <b>POS Tagging: '(토큰, 품사)' 형태로 태깅
>4. <b>Normalization: 표제어 추출
##### 가공
* 감성 분석
    * 사용 라이브러리: nltk.sentiment vader_lexicon
    * 구성
        * 부정 점수(negative)
        * 중립 점수(neutral)
        * 긍정 점수(positive)
        * 종합 점수(compound)
* TF-IDF
    * 사용 라이브러리: sklearn.feature.extraction.text TfidfVectorizer
    * 범위: 1달, 곡당 상위 3개 키워드
* KeyBERT
    * 사용 라이브러리: KeyBERT
    * 범위: 1년, 상위 30개 키워드
#### Album Cover
수집한 앨범 커버 링크를 활용한 색 추출
* 사용 라이브러리: colorthief
* 추출 데이터 처리: HSL 기반 범주화 
## 5. 데이터 분석
### EDA 
#### [1] 팬데믹 이후 트렌드: BPM, Duration, Genre
<p align = 'center'>
    <img src = 'image.png'><br>
</p>

>1. 2019년 팬데믹 이후 BPM은 급속도로 증가, 노래 길이는 급격히 감소<br>
>2. 2019년 이후, 혼자 집에 머무는 시간이 늘어나게 되면서 숏폼 컨텐츠의 인기 증가<br>
>3. 짧은 시간 안에 대중의 귀를 사로잡을 수 있는 빠른 BPM, 곡 길이가 짧은 노래가 인기를 얻음<br>
>4. 단순히 2019년에 인기를 얻은 것으로 그친 것이 아니라, 새로운 트렌드를 이끌어 나감<br>

<p align = 'center'>
    <img src = 'image-1.png'><br>
</p>

*장르의 경우, 모든 연도의 1, 2위가 POP, 랩/힙합이므로 타장르의 변화를 시각화하기 위해 해당 값 제거*<br>

>1. 2019년 이후, 컨트리 장르가 부상하면서 2023년 POP, 랩/힙합을 제외한 장르 중 1등을 차지
>2. MinMaxScaling을 진행한 후 다시 시각화 한 결과, 컨트리 장르가 2019년 이후로 급속도로 부상<br> 
(+) 컨트리 빈도수가 급증한 이유<br>
    1) 코로나19로 인해 사람들이 자택에 머무는 시간이 증가, 재택근무 증가
    2) 틱톡/릴스 등의 숏폼의 인기로 숏폼용 음악 비중 증가
    3) 자극적인 숏폼 노래로 인한 대중의 피로도 증가 
    4) 비교적 담백한 컨트리 음악 수요가 2019년 ~ 2022년 대비 약 58% 증가 

#### [2] 앨범 커버 컬러 트렌드 변화<br>
##### *<b>2년 주기로 밝아지는 컬러 트렌드 & 레드의 강세</b>*

<p align = 'center'>
    <img src = 'image-2.png'><br>
</p>

> 1. 짝수에서 홀수로 넘어가는 겨울에 RGB값이 모두 치솟는 경향
> 2. 전반적으로 R 값이 상위에 위치

<p align = 'center'>
    <img src = 'image-3.png'><br>
</p>

>HSL 기반 텍스트 데이터를 집계한 결과, 'RED'의 비율이 약 36% 차지 <br>

##### *<b>점점 진해지고 있는 컬러 트렌드</b>*

<figure class="half">  
    <img src="image-4.png">  
    <img src="image-5.png"> 
</figure>

>1. 지난 10년간의 RGB값은 시간을 기준으로 기울기는 작다고 볼 수 있으나, 모두 꾸준히 음의 상관관계를 갖고 있다. 
>2. 색상 시각화 결과, 해가 갈수록 진해지고 있는 것을 확인 가능

#### [3] 가사 감성 분석<br>
##### *<b>10년 간 빌보드 HOT100의 긍정/부정 비율</b>*

<figure align = 'center'>  
    <img src="image-6.png">  
</figure>

> 10년 간의 빌보드 HOT100 가사를 감성 분석한 결과, 종합점수 기준 양수인 긍정 비율은 61.2%, 음수인 부정 비율은 38.8%로 긍정적인 노래가 많은 것으로 나타났다. 

##### *<b>월별 감성 종합점수 평균</b>*

<figure align = 'center'>  
    <img src="image-7.png">  
</figure>

> [계절성 확인]
> 1. 겨울(12월)에 가파르게 상승하는 감성 점수
> 2. 가을(10월)에 가파르게 하락하는 감성 점수

##### *<b>순위가 높아질수록 극단적인 노래들</b>*

<figure align = 'center'>  
    <img src="image-8.png">  
</figure>

*10년 간의 빌보드 HOT100 곡들의 순위를 5그룹(1~20, 21~40, ...)으로 클러스터링 한 뒤, 순위 그룹 간 감정 종합 점수 분포 확인*

> 상위 그룹일수록 중립 곡의 분포가 적은 것을 확인 

##### *<b>그럼에도 사랑받는 긍정적인 노래들</b>*

<figure align = 'center'>
    <img src = 'image-9.png'>
</figure>

> 상위 그룹일수록 평균 감성 점수가 높은 것을 확인(평균 감성 점수가 높다 = 긍정적인 노래가 많음)

##### *<b>상위 20개 장르의 평균 감성 점수 비교</b>*

<figure align = 'center'>
    <img src = 'image-10.png'>
</figure>

> 장르와 감성 점수의 관계는 확인할 수 없었다. 장르에 따라붙는 이미지는 편견일 뿐이다. 

#### [3] 가사 TF-IDF 분석<br>

<figure class="half">  
    <img src="image-11.png">  
    <img src="image-12.png"> 
</figure>

> 대부분의 가사에서 TF-IDF 점수가 낮은 일반 단어가 주로 나타났고,높은 점수의 키워드 단어는 매우 적은 수로 나타났다. <br>
> (TF-IDF 점수가 낮은 경우: 다른 곡에도 많이 쓰이는 단어 / TF-IDF 점수가 높은 경우: 해당 곡에만 쓰이는 단어)
### 요인 분석
#### PCA
* 사용 변수 목록
    * Year/Month/Week
    * Rank
    * BPM
    * Duration_sec
    * neu
    * pos
    * compound
    * R, G, B
    * isTop20
* 결과
    |component|설명|
    |---|---|
    |component 1|감정적이고 긍정적인 노래를 구별하는 데 유용|
    |component 2|리듬, 템포 등의 음악적 요소를 반영|
    > 부정적인 감정이나 특정한 주제를 포함하고, 리듬/템포 등과 같은 음악적 요소를 이용한다면 충분히 TOP20 안에 들 것이다. 
#### Feature Importance 
<figure align = 'center'>
    <img src = 'image-13.png'>
</figure>


## 6. 모델링
### 모델링 개요
* 목표: 빌보드 차트 기준 Top20을 Target으로 한 분류 모델 구축
* 사용 데이터 
    * 독립변수: Year, Month, Week, BPM, Duration_sec, R, G, B, compound, Genre, Featuring
    * 종속변수: isTop20 (0: X, 1: O)
* 사용 Feature
    * 사용: Year, Month, Week, Genre, BPM, Duration_sec, R, G, B, Featuring, compound, isTop20
    * 제외: 
        * Rank: 'isTop20'으로 대체
        * Title
        * Artist
        * Lyrics
        * color1, color2, color3: 'R', 'G', 'B'로 대체
        * neg, neu, pos: 'compound'로 대체 
* 성능 평가 지표: F1-score
    > 해당 모델은 타겟 데이터와 아닌 데이터의 비율이 1:4인 불균형 데이터를 사용하는 모델이므로, accuracy가 아닌 f1-score를 성능 평가 지표로 사용
### 모델 비교 & 성능 최적화
* 사용 알고리즘: 계열별로 하나씩 모델 선정
    * 경사하강법 기반: Logistic Regression
    * 확률 기반: Naive Bayes
    * 거리 기반: SVM
    * 트리기반: (기본) Decision Tree, (앙상블) Random Forest

<figure align = 'center'>
    <img src = 'image-14.png'>
</figure>

#### 최종 모델
|항목| |
|---|---|
|Features|전체, abs(compound)|
|Train:Test|8:2|
|Scaler|Standard|
|Class_weight|balanced|subsample|

* 성능
    * 베이스라인 성능: 0.8290
    * 최종 모델 성능: 0.8415

#### 최종 모델 해석
*PDP(Partial Dependence Plot): 부분 의존도*

##### Genre
<figure align = 'center'>
    <img src = 'image-15.png'>
</figure>

##### Week
<figure align = 'center'>
    <img src = 'image-16.png'>
</figure>

##### BPM
<figure align = 'center'>
    <img src = 'image-17.png'>
</figure>

##### Duration
<figure align = 'center'>
    <img src = 'image-18.png'>
</figure>

##### Color: Red
<figure align = 'center'>
    <img src = 'image-19.png'>
</figure>

##### Color: Green
<figure align = 'center'>
    <img src = 'image-20.png'>
</figure>

##### Color: Blue
<figure align = 'center'>
    <img src = 'image-21.png'>
</figure>

<figure align = 'center'>
    <img src = 'image-22.png'>
</figure>

##### compound
<figure align = 'center'>
    <img src = 'image-23.png'>
</figure>

##### Featuring
<figure align = 'center'>
    <img src = 'image-24.png'>
</figure>


## 7. 결론
### 인사이트
#### 지난 10년 간의 음악 트렌드 
* 2019년 이후 변화: BPM 증가, Duration 감소, Genre는 컨트리
* 앨범 커버 컬러 트렌드: 2년 주기로 밝아짐, 레드 강세, 전반적으로 어두워지는 경향
* 가사 감성: 긍정 강세, 계절성(가을 - 감성 점수 하락, 겨울 - 감성 점수 증가)
* 가사 키워드: 비슷한 단어를 반복적으로 쓰는 경향

#### 분류 모델에서 얻은 인사이트
> 빌보드 Top20에 들기 위해서는 BPM, Compound, Genre가 크게 작용한다. 

### 결론: 제안 타이틀 곡 컨셉
* 장르: 컨트리풍의 POP
* 발매 시기: 연초, 첫 번째 주
* BPM: 100 ~ 120
* Duration: 210 (약 3분 30초)
* 앨범 커버 RGB: (120, 100, 168)
* 감성: 매우 긍정적
* 피쳐링: 없이 진행
<figure align = 'center'>
    <img src = 'image-25.png'>
</figure>

## 8. 보완점, 발전 방향
* 추가 항목 수집: spotify API에 있는 노래 분위기 데이터를 추가로 수집하여 관련 분석을 진행한다면, 더욱 다차원적인 분석이 가능할 것으로 기대
* 다장르 노래 분석: 장르의 경우, 해당 프로젝트에서는 복잡성을 줄이기 위해 한 곡당 하나의 장르만 파악했지만, 여러 개의 장르를 추출하여 분석한다면 명확한 분석이 될 것으로 기대(현대곡의 경우, 단일 장르가 아닌 경우가 대부분)
* 팬덤 데이터 적용: 가수의 영향력을 반영한 분석, 해당 팬층의 당시 반응이나 sns 반응 등을 분석한다면 더욱 공교한 비즈니스 전략이 될 것으로 기대