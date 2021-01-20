<h1 align="center"><strong>신용카드 사용 내역 EDA 프로젝트</strong></h3>

<p align="center"><img src="https://user-images.githubusercontent.com/72811950/105155279-bf002800-5b4d-11eb-8af9-b2f5bc72215f.jpg"></p>

## 1. 목적
- 2년 동안(2016년 8월 ~ 2018년 07월) 한국의 약 2000여 개 상점의 신용카드 거래내역을 이용하여 데이터 시각화 및 인사이트 발굴
- 신용카드 거래내역 데이터는 DACON에서 가져온 것으로 https://dacon.io/competitions/official/42473/data/ 에서 확인하실 수 있습니다.

## 2. 설치
Requirements
```
* Python 3.6+
```
Installation
```
pip install numpy
pip install pandas
pip install matplotlib
pip install seaborn
pip install plotly
pip install tqdm
```
## 3. EDA(Exploratory Data Analysis)
0) 패키지 불러오기
```
from matplotlib import font_manager, rc
from plotly.subplots import make_subplots
from tqdm import tqdm
import matplotlib.pyplot as plt 
import seaborn as sns 
import plotly.graph_objs as go
import plotly.express as px
import numpy as np
import pandas as pd
%matplotlib inline

plt.rcParams['axes.unicode_minus'] = False
f_path = "/Library/Fonts/Arial Unicode.ttf"
font_name = font_manager.FontProperties(fname=f_path).get_name()
rc('font', family=font_name)
```
1) 데이터 확인
- 데이터 정보  
변수 8개 관측치는 약 330만개로 관측치에 비해 변수가 적은편
```
store_id :각 파일에서의 상점 고유 번호
date : 거래 일자
time :거래 시간
card_id :카드 번호의 hash 값
amount :매출액 (여기서 매출액은 KRW가 아님)
installments : 할부개월수. 일시불은 빈 문자열
days_of_week : 요일, 월요일이 0, 일요일은 6
holiday : 0이면 공휴일 아님, 1이면 공휴일
```  

- 데이터 유니크 값 개수  
1775개의 상점에서 922522개의 신용카드가 조사됨을 확인  
<img src="https://user-images.githubusercontent.com/72811950/105183034-1368cf00-5b71-11eb-8ccd-518850ebe4e9.png" width="200" height="300"></img>  
![newplot (1)](https://user-images.githubusercontent.com/72811950/105183157-2f6c7080-5b71-11eb-85a5-e4011cf31d50.png)

2) 전처리
- installments 컬럼에 3345936개의 결측치 있는 것을 확인
![newplot](https://user-images.githubusercontent.com/72811950/105180715-4198df80-5b6e-11eb-9c30-073937cf263d.png)

3) 시간에 따른 매출 분석


4) 할부 내역 분석


5) 거래 취소내역 분석



## 4. 결과

## 5. 후기
처음 하게 된 프로젝트였다.  
그래서 복잡한 데이터를 선택하는 것보다는 비교적 변수의 개수가 적은 데이터를 선택하는 것이 좋다고 생각했다.  
만약 변수 개수가 더 많은 데이터를 선택했다면 다양한 인사이트를 도출하고 시각화 했을 때 더 많은 그래프를 볼 수 있었을 것이다.  
좀 더 데이터를 시각화하는데 익숙해지면 신용카드 사용지역과 상점의 유형이 추가된 데이터에 도전해 보아야겠다.

