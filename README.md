<h1 align="center"><strong>신용카드 사용 내역 EDA 프로젝트</strong></h3>

<p align="center"><img src="https://user-images.githubusercontent.com/72811950/105155279-bf002800-5b4d-11eb-8af9-b2f5bc72215f.jpg"></p>

## 1. 목적
- 2년 동안(2016년 8월 ~ 2018년 07월) 한국의 약 2000여 개 상점의 신용카드 거래내역을 이용하여 데이터 시각화 및 인사이트 발굴
- 신용카드 거래내역 데이터는 DACON에서 가져온 것으로 https://dacon.io/competitions/official/42473/data/ 에서 확인하실 수 있습니다.

## 2. 설치
### Requirements
```
* Python 3.6+
```
### Installation
```
pip install numpy
pip install pandas
pip install matplotlib
pip install seaborn
pip install plotly
pip install tqdm
```
## 3. EDA(Exploratory Data Analysis)
### 3.1 패키지 불러오기
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
### 3.2 데이터 확인  
#### 1) 데이터 정보  
- 변수 8개 관측치는 3362796개로 관측치에 비해 변수가 적은편
```
store_id :각 파일에서의 상점 고유 번호
date : 거래 일자
time : 거래 시간
card_id : 카드 번호의 hash 값
amount : 매출액(여기서 매출액은 KRW가 아님)
installments : 할부 개월 수. 일시불은 빈 문자열
days_of_week : 요일, 월요일이 0, 일요일은 6
holiday : 0은 공휴일 아닌 날, 1은 공휴일
```  
#### 2) 데이터 유니크 값 개수  
- 1775개의 상점에서 922522개의 신용카드가 조사됨  
  
  <img src="https://user-images.githubusercontent.com/72811950/105183034-1368cf00-5b71-11eb-8ccd-518850ebe4e9.png" width="240" height="300"></img>  
3) 결측치 확인  
- installments 컬럼에 3345936개의 결측치 있음  
<img src="https://user-images.githubusercontent.com/72811950/105180715-4198df80-5b6e-11eb-9c30-073937cf263d.png" width="700" height="500"></img>

### 3.3 전처리  
1) 결측치 처리  
- installments에서 결측치는 일시불을 의미하므로 1로 채움  
2) 이상치 확인 및 처리  
- amount는 환불 때문에 -값이 존재. 분석을 위해 환불 건은 제거.  
- amount컬럼에서 전체 상점 일일 총 매출의 1/4을 차지할 정도로 큰 이상치가 있음.  
  이상치라고 여겨지는 대부분의 결제건이 높은 금액의 물건을 판매하는 한 두 상점의 매출이어서 그 상점들의 매출을 다 제거하는 것이 바람직하지 않다고 생각하여 격차가 매우 큰 이상치 한 개만 제거하기로 함.  

### 3.4 시간에 따른 매출 분석  
1) 월별 총 매출과 결제횟수
```
monthly_sales = credit.groupby("year-month")["amount"].sum().reset_index(name="monthly_total_amount")
monthly_count = credit.groupby("year-month").size().reset_index(name="monthly_count") 

colors = ["#316395"] * 24
colors[4] = 'crimson'
colors[16] = 'crimson'

fig = make_subplots(rows=2, cols=1, shared_xaxes=True, row_heights = [0.4, 0.6], vertical_spacing=0.02)

fig.append_trace(go.Scatter(
    x=monthly_count["year-month"],
    y=monthly_count["monthly_count"],
    marker_color="red",
    name="결제 횟수"
), row=1, col=1)

fig.append_trace(go.Bar(
    x=monthly_sales["year-month"],
    y=monthly_sales["monthly_total_amount"],
    marker_color=colors,
    name="총 매출액"
), row=2, col=1)

fig.update_layout(height=300*2, width=260*3, title_text="월별 총 매출과 결제 횟수", font_size=12)
fig.update_xaxes(title_text="월", row=2, col=1)
fig.update_yaxes(title_text="결제 횟수", row=1, col=1)
fig.update_yaxes(title_text="총 매출액", row=2, col=1)
fig.show()
```  
<img src="https://user-images.githubusercontent.com/72811950/105197959-84fc4980-5b80-11eb-9525-a58570afd938.png" width="800" height="580"></img>
- 12월에는 매출액이 증가하는 것을 볼 수 있다. 연말이라 사람들의 소비가 늘어난 것으로 보임.  
  
2) 2017년 시즌별 매출(월별로 확인)
```
credit_2017 = credit[credit["year"] ==2017]
seasonal_sales = credit_2017.groupby("month")["amount"].sum().reset_index(name="seasonal_sales")

colors = ["#316395"] * 12
colors[11] = 'crimson'

fig = go.Figure()
data = go.Bar(x=seasonal_sales["month"], y=seasonal_sales["seasonal_sales"], marker_color=colors)
fig.add_trace(data)
fig.update_layout(title='시즌별 매출',
                  xaxis_title='월',
                  yaxis_title='총 매출액',
                  font_size=12)
fig.show()
```  
<img src="https://user-images.githubusercontent.com/72811950/105198719-49ae4a80-5b81-11eb-8210-68162ad201da.png" width="700" height="500"></img>  
- 2년간의 정보가 있지만 2016년 하반기와 2018년 상반기 데이터가 합쳐진 것으로 주변상황(물가, 소비자 심리 등)을 고려했을 때  
  매출 비교가 제대로 이뤄지지 않을 것으로 판단 2017년 한 해의 데이터로만 분석 
- 겨울에는 신용카드 사용이 줄어듦. 하지만 12월은 연말이라 사람들의 소비가 늘어난 것으로 보임.  
  
3) 요일별 매출
```
credit_mon = credit[credit['days_of_week']==0]
credit_tue = credit[credit['days_of_week']==1]
credit_wed = credit[credit['days_of_week']==2]
credit_thu = credit[credit['days_of_week']==3]
credit_fri = credit[credit['days_of_week']==4]
credit_sat = credit[credit['days_of_week']==5]
credit_sun = credit[credit['days_of_week']==6]

monday_amount = credit_mon["amount"].sum()
tuesday_amount = credit_tue["amount"].sum()
wednesday_amount = credit_wed["amount"].sum()
thursday_amount = credit_thu["amount"].sum()
firday_amount = credit_fri["amount"].sum()
saturday_amount = credit_sat["amount"].sum()
sunday_amount = credit_sun["amount"].sum()

week_amount_data = {"요일":["월요일", "화요일", "수요일", "목요일", "금요일", "토요일", "일요일"],\
        "총매출":[monday_amount, tuesday_amount, wednesday_amount, thursday_amount, firday_amount, saturday_amount, sunday_amount]}
weekly_amount_df = pd.DataFrame(week_amount_data)
weekly_amount_df

fig = go.Figure()
data = go.Scatter(x=weekly_amount_df["요일"], y=weekly_amount_df["총매출"], marker_color="RED")
fig.add_trace(data)
fig.update_layout(title='요일별 매출',
                  xaxis_title='요일',
                  yaxis_title='매출액',
                  font_size=12)
fig.show()
```  
<img src="https://user-images.githubusercontent.com/72811950/105201979-d27ab580-5b84-11eb-8dfe-ca9fd6b963d4.png" width="700" height="500"></img> 
- 토요일의 매출이 가장 높고 다음으로 금요일이 높았다. 매출이 가장 적은 요일은 일요일이었다.  
  
4) 요일별 시간대 매출과 한 건당 결제 금액
```
monday_hourly_amount = credit_mon.groupby("hour")["amount"].sum().reset_index()
tuesday_hourly_amount = credit_tue.groupby("hour")["amount"].sum().reset_index()
wednsday_hourly_amount = credit_wed.groupby("hour")["amount"].sum().reset_index()
thursday_hourly_amount = credit_thu.groupby("hour")["amount"].sum().reset_index()
friday_hourly_amount = credit_fri.groupby("hour")["amount"].sum().reset_index()
saturday_hourly_amount = credit_sat.groupby("hour")["amount"].sum().reset_index()
sunday_hourly_amount = credit_sun.groupby("hour")["amount"].sum().reset_index()

weekly_mean_slaes1 = credit_mon.groupby('hour')['amount'].mean().reset_index(name="weekly_mean_slaes1")
weekly_mean_slaes2 = credit_tue.groupby('hour')['amount'].mean().reset_index(name="weekly_mean_slaes2")
weekly_mean_slaes3 = credit_wed.groupby('hour')['amount'].mean().reset_index(name="weekly_mean_slaes3")
weekly_mean_slaes4 = credit_thu.groupby('hour')['amount'].mean().reset_index(name="weekly_mean_slaes4")
weekly_mean_slaes5 = credit_fri.groupby('hour')['amount'].mean().reset_index(name="weekly_mean_slaes5")
weekly_mean_slaes6 = credit_sat.groupby('hour')['amount'].mean().reset_index(name="weekly_mean_slaes6")
weekly_mean_slaes7 = credit_sun.groupby('hour')['amount'].mean().reset_index(name="weekly_mean_slaes7")

fig = make_subplots(
    rows=2, cols=1, shared_xaxes=True, vertical_spacing=0.02,
    row_heights = [0.5, 0.5],
)

fig.add_trace(go.Scatter(x=monday_hourly_amount["hour"][::-1], y=monday_hourly_amount["amount"][::-1], name="월요일", marker_color="black"), row=1, col=1)
fig.add_trace(go.Scatter(x=tuesday_hourly_amount["hour"][::-1], y=tuesday_hourly_amount["amount"][::-1], name="화요일", marker_color="yellow"), row=1, col=1)
fig.add_trace(go.Scatter(x=wednsday_hourly_amount["hour"][::-1], y=wednsday_hourly_amount["amount"][::-1], name="수요일", marker_color="green"), row=1, col=1)
fig.add_trace(go.Scatter(x=thursday_hourly_amount["hour"][::-1], y=thursday_hourly_amount["amount"][::-1], name="목요일", marker_color="blue"), row=1, col=1)
fig.add_trace(go.Scatter(x=friday_hourly_amount["hour"][::-1], y=friday_hourly_amount["amount"][::-1], name="금요일", marker_color="purple"), row=1, col=1)
fig.add_trace(go.Scatter(x=saturday_hourly_amount["hour"][::-1], y=saturday_hourly_amount["amount"][::-1], name="토요일", marker_color="orange"), row=1, col=1)
fig.add_trace(go.Scatter(x=sunday_hourly_amount["hour"][::-1], y=sunday_hourly_amount["amount"][::-1], name="일요일", marker_color="red"), row=1, col=1)

fig.add_trace(go.Scatter(x=weekly_mean_slaes1["hour"][::-1], y=weekly_mean_slaes1["weekly_mean_slaes1"][::-1], showlegend=False, marker_color="black"),row=2, col=1)
fig.add_trace(go.Scatter(x=weekly_mean_slaes2["hour"][::-1], y=weekly_mean_slaes2["weekly_mean_slaes2"][::-1], showlegend=False, marker_color="yellow"),row=2, col=1)
fig.add_trace(go.Scatter(x=weekly_mean_slaes3["hour"][::-1], y=weekly_mean_slaes3["weekly_mean_slaes3"][::-1], showlegend=False, marker_color="green"),row=2, col=1)
fig.add_trace(go.Scatter(x=weekly_mean_slaes4["hour"][::-1], y=weekly_mean_slaes4["weekly_mean_slaes4"][::-1], showlegend=False, marker_color="blue"),row=2, col=1)
fig.add_trace(go.Scatter(x=weekly_mean_slaes5["hour"][::-1], y=weekly_mean_slaes5["weekly_mean_slaes5"][::-1], showlegend=False, marker_color="purple"),row=2, col=1)
fig.add_trace(go.Scatter(x=weekly_mean_slaes6["hour"][::-1], y=weekly_mean_slaes6["weekly_mean_slaes6"][::-1], showlegend=False, marker_color="orange"),row=2, col=1)
fig.add_trace(go.Scatter(x=weekly_mean_slaes7["hour"][::-1], y=weekly_mean_slaes7["weekly_mean_slaes7"][::-1], showlegend=False, marker_color="red"),row=2, col=1)

fig.update_layout(height=300*2, width=260*3, title_text="시간대별 매출액과 한 건당 결제금액", showlegend=True, font_size=12)
fig.update_xaxes(title_text="시간", row=2, col=1)
fig.update_yaxes(title_text="총 매출", row=1, col=1)
fig.update_yaxes(title_text="한 건당 결제금액", row=2, col=1)

fig.show()
```  
<img src="https://user-images.githubusercontent.com/72811950/105202679-985de380-5b85-11eb-9625-a4558ae1b41c.png" width="800" height="580"></img>
- 12시와 18시에 카드 하나당 결제 금액이 낮아지는 것으로 보아 식사 후 각자 계산 하는 경우가 많은 것으로 추측
- 10시와 15시에는 카드 하나당 결제 금액이 높아진다. 모임이나 쇼핑등으로 한 번에 큰 금액이 결제되는 것으로 추측
- 20시에는 총 매출액이 가장 높고 한 건당 결제되는 금액도 높다. 술자리 때문이 아닐까 추측
- 다른 요일에 비해 금요일은 늦은 시간까지 매출이 높다.
- 일요일은 평일에 비해 일찍 매출이 떨어지기 시작한다.(19시에 매출 정점을 찍고 떨어지기 시작한다.)
- 평일에는 12시와 20에 매출이 상승하다가 떨어지는 반면 토요일과 일요일은 13시와 19시에 매출이 상승했다 떨어진다.  
  
5) 공휴일과 평일 시간대 매출 비교
```
non_holyday = credit[credit['holyday']==0]
holyday = credit[credit['holyday']==1]
non_holyday_amount_mean = non_holyday.groupby('hour')['amount'].sum().reset_index()
holyday_amount_mean = holyday.groupby('hour')['amount'].sum().reset_index()

fig = go.Figure()
fig.add_trace(go.Scatter(x=non_holyday_amount_mean["hour"], y=non_holyday_amount_mean["amount"],\
                         name="비공휴일", marker_color="blue"))
fig.add_trace(go.Scatter(x=holyday_amount_mean["hour"], y=holyday_amount_mean["amount"],\
                         name="공휴일", marker_color="RED"))            
fig.update_layout(title='시간대 평일 VS 휴일 평균 매출',
                  xaxis_title='시간',
                  yaxis_title='평균 매출',
                  font_size=12)
fig.show()
```  
<img src="https://user-images.githubusercontent.com/72811950/105204185-3d2cf080-5b87-11eb-9ef4-5c02b1129877.png" width="700" height="500"></img>
- 시간대 매출 확인 결과 공휴일보다 평일에 사람들의 소비가 높았다.


### 4) 할부 내역 분석


### 5) 거래 취소내역 분석



## 4. 결과

## 5. 후기
처음 하게 된 프로젝트였다.  
그래서 복잡한 데이터를 선택하는 것보다는 비교적 변수의 개수가 적은 데이터를 선택하는 것이 좋다고 생각했다.  
만약 변수 개수가 더 많은 데이터를 선택했다면 다양한 인사이트를 도출하고 시각화했을 때 더 많은 그래프를 볼 수 있었을 것이다.  
좀 더 데이터를 시각화하는데 익숙해지면 신용카드 사용지역과 상점의 유형이 추가된 데이터에 도전해 보아야겠다.

