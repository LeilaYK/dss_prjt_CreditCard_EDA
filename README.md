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
1) 패키지 불러오기
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
2) 전처리


3) 시간에 따른 매출 분석


4) 할부 내역 분석


5) 거래 취소내역 분석



## 4. 결과

## 5. 후기
