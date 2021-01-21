<h1 align="center"><strong>신용카드 사용 내역 EDA 프로젝트</strong></h3>

<p align="center"><img src="https://user-images.githubusercontent.com/72811950/105155279-bf002800-5b4d-11eb-8af9-b2f5bc72215f.jpg"></p>

## 1. 목적
- 한국의 약 2000여 개 상점의 2년 동안의 신용카드 거래내역을 이용하여 데이터 시각화 및 인사이트 발굴.
- 신용카드 거래내역 데이터는 DACON에서 가져온 것으로 https://dacon.io/competitions/official/42473/data/  
  에서 확인하실 수 있습니다.

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
- 변수 8개 관측치는 3362796개로 관측치에 비해 변수가 적은편이다.
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
- 1775개의 상점에서 922522개의 신용카드가 조사됨.  
  
  <img src="https://user-images.githubusercontent.com/72811950/105183034-1368cf00-5b71-11eb-8ccd-518850ebe4e9.png" width="240" height="300"></img>  
#### 3) 결측치 확인  
- installments 컬럼에 3345936개의 결측치 확인.  
<img src="https://user-images.githubusercontent.com/72811950/105180715-4198df80-5b6e-11eb-9c30-073937cf263d.png" width="700" height="500"></img>

### 3.3 전처리  
#### 1) 결측치 처리  
- installments에서 결측치는 일시불을 의미하므로 1로 채움.  
#### 2) 이상치 확인 및 처리  
- amount는 환불 때문에 값이 존재. 분석을 위해 환불 건은 제거.  
- amount컬럼에서 전체 상점 일일 총 매출의 1/4을 차지할 정도로 큰 이상치가 있음.  
  이상치라고 여겨지는 대부분의 결제건이 높은 금액의 물건을 판매하는 한 두 상점의 매출이어서 그 상점들의 매출을 다 제거하는 것이 바람직하지 않다고 생각하여 격차가 매우 큰 이상치 한 개만 제거하기로 함.  

### 3.4 시간에 따른 매출 분석  
#### 1) 월별 총 매출과 결제횟수
```
# 2년동안의 월별 총 매출
monthly_sales = credit.groupby("year-month")["amount"].sum().reset_index(name="monthly_total_amount")
# 2년동안의 월별 결제 횟수
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
- 결제 횟수와 매출액이 비례하는 것을 알 수 있다.
- 시간의 흐름에 따라 결제 횟수와 매출액이 상승하는 모양을 보인다.
- 12월에는 매출이 증가하는 것을 볼 수 있다. 연말이라 사람들의 소비가 늘어난 것으로 보임.  
  
#### 2) 2017년 시즌별 매출(월별로 확인)
```
# 2017년도 데이터만 추출
credit_2017 = credit[credit["year"]==2017]
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
  매출 비교가 제대로 이뤄지지 않을 것으로 판단 2017년 한 해의 데이터로만 분석. 
- 1월과 2월 겨울에는 매출이 낮았다가 봄이 되면서 매출이 계속 상승한다.
- 12월은 연말이라 사람들의 소비가 증가하는 것으로 보인다.
  
#### 3) 요일별 매출
```
# 요일별로 묶기
credit_mon = credit[credit['days_of_week']==0]
credit_tue = credit[credit['days_of_week']==1]
credit_wed = credit[credit['days_of_week']==2]
credit_thu = credit[credit['days_of_week']==3]
credit_fri = credit[credit['days_of_week']==4]
credit_sat = credit[credit['days_of_week']==5]
credit_sun = credit[credit['days_of_week']==6]
# 요일별 매출
monday_amount = credit_mon["amount"].sum()
tuesday_amount = credit_tue["amount"].sum()
wednesday_amount = credit_wed["amount"].sum()
thursday_amount = credit_thu["amount"].sum()
firday_amount = credit_fri["amount"].sum()
saturday_amount = credit_sat["amount"].sum()
sunday_amount = credit_sun["amount"].sum()
# 요일별 매출 데이터프레임
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
- 토요일 매출이 가장 높고 다음으로 금요일이 높았다. 매출이 가장 적은 요일은 일요일이었다.  
  
#### 4) 요일별 시간대 매출과 한 건당 결제 금액
```
# 요일별 시간대 총 매출
monday_hourly_amount = credit_mon.groupby("hour")["amount"].sum().reset_index()
tuesday_hourly_amount = credit_tue.groupby("hour")["amount"].sum().reset_index()
wednsday_hourly_amount = credit_wed.groupby("hour")["amount"].sum().reset_index()
thursday_hourly_amount = credit_thu.groupby("hour")["amount"].sum().reset_index()
friday_hourly_amount = credit_fri.groupby("hour")["amount"].sum().reset_index()
saturday_hourly_amount = credit_sat.groupby("hour")["amount"].sum().reset_index()
sunday_hourly_amount = credit_sun.groupby("hour")["amount"].sum().reset_index()
# 요일별 시간대 한 건당 매출
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
- 점심 시간인 12시 저녁시간인 18시부터 20시까지 총 매출이 상승한다.
- 12시와 18시에 총 매출은 상승하지만 카드 하나당 결제 금액이 낮아지는 것으로 보아 식사 후 각자 계산 하는 경우가 많은 것으로 추측
- 10시와 15시에는 카드 하나당 결제 금액이 높아진다. 모임이나 쇼핑등으로 한 번에 큰 금액이 결제되는 것으로 추측
- 20시에는 총 매출액이 가장 높고 한 건당 결제되는 금액도 높다.
- 20시 이후의 총 매출을 보면 다른요일에 비해 금요일 매출이 높다.
- 일요일은 평일에 비해 일찍 매출이 떨어지기 시작한다.(19시에 매출 정점을 찍고 떨어지기 시작한다.)
- 평일에는 12시와 20에 매출이 상승하다가 떨어지는 반면 토요일과 일요일은 13시와 19시에 매출이 상승했다 떨어진다.  
  
#### 5) 공휴일과 평일 시간대 매출 비교
```
# 공휴일과 평일로 데이터 나누기
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
- 시간대 매출 확인 결과 공휴일보다 평일에 사람들의 소비가 많았다.  
  
#### 6) 휴일 전날과 아닌날의 매출 비교
```
# 공휴일 데이터만 추출
holyday = credit[credit['holyday']==1]
holiday_list = holyday.date.unique()
# 공휴일에서 하루를 빼서 공휴일 전날의 날짜를 리스트에 
d = np.timedelta64(1, 'D')
before_list = holiday_list - d
before_holyday = credit[(credit["date"].isin(before_list))|(credit['days_of_week']==5)]
non_before_holyday = credit[~(credit["date"].isin(before_list))&(credit['days_of_week']!=5)]
b_holyday = before_holyday.groupby('hour')['amount'].sum().reset_index()
non_b_holyday = non_before_holyday.groupby('hour')['amount'].sum().reset_index()

fig = go.Figure()
fig.add_trace(go.Scatter(x=b_holyday["hour"], y=b_holyday["amount"],\
                         name="다음날이 휴일인 날", marker_color="red"))
fig.add_trace(go.Scatter(x=non_b_holyday["hour"], y=non_b_holyday["amount"],\
                         name="다음날이 휴일이 아닌 날", marker_color="blue"))            
fig.update_layout(title='공휴일 전날과 아닌 날의 매출 비교',
                  xaxis_title='시간',
                  yaxis_title='총 매출',
                  font_size=12)
fig.show()
```  
<img src="https://user-images.githubusercontent.com/72811950/105205061-29ce5500-5b88-11eb-8399-549cf7fd24c4.png" width="800" height="500"></img>
- 휴일 전날보다 평일에 사람들의 소비가 많이 이뤄진다.  
  
#### 7) 일일 구매 횟수와 매출
```
# 일별 구매횟수
daily_count = credit.groupby("date").size().reset_index(name="daily_count")
# 일별 매출합
daily_amount = credit.groupby("date")["amount"].sum().reset_index(name="daily_amount")

fig = make_subplots(rows=2, cols=1, shared_xaxes=True, vertical_spacing=0.02, row_heights = [0.5, 0.5])

fig.add_trace(go.Scatter(x=daily_count["date"][::-1], y=daily_count["daily_count"][::-1],\
                         name="구매 횟수", marker_color="red"), row=1, col=1)

fig.add_trace(go.Scatter(x=daily_amount["date"][::-1], y=daily_amount["daily_amount"][::-1],\
                         name="매출액", marker_color="blue"),row=2, col=1)

fig.update_layout(height=300*2, width=260*3, title_text="일일 구매 횟수와 매출액", font_size=12)
fig.update_xaxes(title_text="일", row=2, col=1)
fig.update_yaxes(title_text="구매 횟수", row=1, col=1)
fig.update_yaxes(title_text="총 매출액", row=2, col=1)

fig.show()
```  
<img src="https://user-images.githubusercontent.com/72811950/105205381-8e89af80-5b88-11eb-9e26-78f4239dba5f.png" width="800" height="580"></img>
- 일일 구매 횟수와 매출 두 그래프가 비슷한 형태를 보이고 있다. 
- 구매횟수와 매출이 떨어지는 날을 확인해보니 휴일이었다.  
  (2016-09-15, 2017-10-04 : 추석, 2017-01-28, 2018-02-16 : 설)
- 추석과 설 당일에는 휴무인 상점이 많아 거래가 줄어든 것으로 보인다.  

### 3.4 할부 내역 분석  
#### 1) 할부별 빈도수
```
# 할부 데이터만 추출
installments_df = credit[(credit["installments"] > 1)]

fig = px.histogram(installments_df, x="installments", color_discrete_sequence=['orange'])
fig.update_layout(title_text="사람들은 몇 개월 할부를 가장 많이 할까?", font_size=12)
fig.update_xaxes(title_text="할부 개월수")
fig.update_yaxes(title_text="빈도수")
fig.show()
```  
<img src="https://user-images.githubusercontent.com/72811950/105207154-86327400-5b8a-11eb-9bfc-bf998ec2ff3e.png" width="700" height="500"></img>
- 3개월 할부가 가장 많았고 그 다음으로 2개월, 5개월 할부가 많았다.  
  
#### 2) 요일별 할부 결제 횟수
```
installments_count_by_day = installments_df.groupby("days_of_week").size().reset_index(name="빈도수")
installments_count_by_day["days_of_week"] = ["월요일", "화요일", "수요일", "목요일", "금요일", "토요일", "일요일"]

fig = go.Figure()
fig.add_trace(go.Scatter(x=installments_count_by_day["days_of_week"], y=installments_count_by_day["빈도수"],\
                  marker_color="red"))
fig.update_layout(title='요일별 할부결제 총 횟수',
                  xaxis_title='요일',
                  yaxis_title='결제 횟수',
                  font_size=12)
fig.show()
```  
<img src="https://user-images.githubusercontent.com/72811950/105209364-170a4f00-5b8d-11eb-93c8-251b14a733ad.png" width="700" height="500"></img>  
- 토요일 다음으로 금요일, 화요일 순으로 할부 결제가 많았고 일요일이 할부 결제가 가장 적었다.
- 위의 요일별 총 매출 그래프와 비교했을 때 토요일과 금요일은 매출이 높은 만큼 할부도 많다고 생각되는데 화요일의 경우는 좀 다르다.  
  화요일의 총 매출은 일요일, 월요일 다음으로 총 매출이 낮은 요일인데 할부 횟수는 세 번째로 많다.
  
#### 3) 일시불과 할부에 따른 시간대 평균 결제금액 비교
```
# 일시불과 할부 데이터 분리
non_installments = credit[credit['installments']==1]
installments = credit[credit['installments']!=1]

hourly_non_installments_mean = non_installments.groupby('hour')['amount'].mean().reset_index(name="non_installments_amount_mean")
hourly_installments_mean = installments.groupby('hour')['amount'].mean().reset_index(name="installments_amount_mean")

fig = go.Figure()
fig.add_trace(go.Scatter(x=hourly_non_installments_mean["hour"], y=hourly_non_installments_mean["non_installments_amount_mean"],\
                         name="일시불", marker_color="blue"))
fig.add_trace(go.Scatter(x=hourly_installments_mean["hour"], y=hourly_installments_mean["installments_amount_mean"],\
                         name="할부", marker_color="red"))            
fig.update_layout(title='시간대별 할부 VS 일시불 평균 결제금액',
                  xaxis_title='시간',
                  yaxis_title='평균 금액',
                  font_size=12)
fig.show()
```  
<img src="https://user-images.githubusercontent.com/72811950/105210595-951b2580-5b8e-11eb-8166-bae63cc0ca0c.png" width="700" height="500"></img>
- 일시불일 때와 할부를 했을 때 두 경우를 나누어 매출액을 시간 별 평균 비교.
- 할부를 했을 때의 매출액이 일시불일 때의 매출액보다 어느 시간대에서든 크게 상회하는 것을 볼 수 있음.  
  이를 통해 큰 금액을 결제할 때 할부를 선택한다는 것을 알 수 있음.
- 일시불일 때는 어떤 시간대이든 매출액이 낮게 유지되는 반면 할부를 했을 때는 시간별 매출액의 변동이 큼.  
  일시불로 결제되는 금액은 상대적으로 한정적이고 적은 금액임을 알 수 있음.

### 3.5 거래 취소내역 분석  
- 결제 취소건을 분석하기 위해 다시 원래 데이터를 가져옴.
- 거래 취소내역은 총 33832건.
  
#### 1) 시간대별 거래 취소 횟수
```
# 거래 취소한 내역만 추출
cancel_df = credit[credit['amount']<0]
cancel_df_hour = cancel_df.groupby('hour').size().reset_index(name='hour_size')

fig = go.Figure()
fig.add_trace(go.Scatter(x=cancel_df_hour["hour"], y=cancel_df_hour["hour_size"],\
                  marker_color="red"))
fig.update_layout(title='시간대별 거래취소횟수',
                  xaxis_title='시간',
                  yaxis_title='거래 취소 횟수',
                  font_size=12)
fig.show()
```  
<img src="https://user-images.githubusercontent.com/72811950/105211247-605b9e00-5b8f-11eb-84b3-98d8c18b7a59.png" width="700" height="500"></img>
- 주로 12시에서 20시 사이에 거래 취소가 발생.
- 12시, 19시에 거래취소가 다른 시간대에 비해 많이 일어나는데 거래가 많이 이뤄지는 시간대이기 때문에 거래취소 또한 많이 일어난다고 볼 수 있음. 

  
#### 2) 시간대별 거래 취소율
```
cancel_ratio = pd.merge(cancel_df_hour, credit.groupby('hour').size().reset_index())
cancel_ratio.columns = ['hour','cancel_size','total_size']
cancel_ratio['cancel_ratio'] = cancel_ratio.cancel_size/cancel_ratio.total_size * 100

fig = go.Figure()
fig.add_trace(go.Scatter(x=cancel_ratio["hour"], y=cancel_ratio["cancel_ratio"],\
                  marker_color="red"))
fig.update_layout(title='시간대별 거래취소율',
                  xaxis_title='시간',
                  yaxis_title='거래 취소율(%)',
                  font_size=12)
fig.show()
```  
<img src="https://user-images.githubusercontent.com/72811950/105212719-29868780-5b91-11eb-87b7-2315e2423533.png" width="700" height="500"></img>
- 거래 취소율을 보면 아침 시간대에서 비율이 높은 것을 확인할 수 있음.  
  하지만 취소율의 최고와 최저가 0.8%밖에 차이가 나지 않아 취소율은 어느시간대나 비슷하다고 할 수 있음.  
  다만, 아침에 취소율이 높은 이유는 거래 횟수가 얼마 되지 않는데 1건이라도 취소가 늘어나면 비율이 급격하게 증가하기 때문이 아닌가 추측.  
    
#### 3) 거래취소가 많이 일어나는 상점의 거래취소 횟수
```
store_cancel = cancel_df.groupby('store_id').size().reset_index(name="cancel_size")
store_cancel = store_cancel.sort_values(by='cancel_size',ascending=False).head(10)
store_cancel = store_cancel.reset_index(drop=True)
store_cancel["store_id"] = ["161번", "1024번", "0번", "221번", "942번", "958번", "428번", "1073번", "356번", "528번"]

fig = go.Figure()
fig.add_trace(go.Bar(x=store_cancel["store_id"], y=store_cancel["cancel_size"], marker_color="#316395"))
fig.update_layout(title='거래 취소 횟수 상위 10개 상점',
                  xaxis_title='상점 ID',
                  yaxis_title='거래 취소 횟수',
                  font_size=12)
fig.show()
```  
<img src="https://user-images.githubusercontent.com/72811950/105213773-76b72900-5b92-11eb-8e97-de8ea915ba8b.png" width="700" height="500"></img>
- 161번, 1024번 상점은 다른 상점들에 비해서 거래 취소 건수가 많음.  
  거래 취소가 많다는 것은 거래 수는 많아 보이지만 실제 매출액으로 계산될 거래 수는 적어지는 것이기 때문에  
  거래 취소가 많은 상점은 더 자세히 살펴볼 필요가 있음.  
    
#### 4) 거래취소가 많이 일어나는 상점의 거래취소율
```
store_cancel = cancel_df.groupby('store_id').size().reset_index(name='cancel_size')
store_cancel = pd.merge(store_cancel, credit.groupby('store_id').size().reset_index())
store_cancel.columns = ['store_id','cancel_size','total_size']
store_cancel['cancel_ratio'] = store_cancel.cancel_size/store_cancel.total_size *100
store_cancel = store_cancel.sort_values(by='cancel_ratio',ascending=False).reset_index(drop=True)
store_cancel_ratio = store_cancel[(store_cancel["store_id"] == 161)|(store_cancel["store_id"] == 1024)|\
                                 (store_cancel["store_id"] == 0)|(store_cancel["store_id"] == 221)|\
                                 (store_cancel["store_id"] == 942)|(store_cancel["store_id"] == 958)|\
                                 (store_cancel["store_id"] == 428)|(store_cancel["store_id"] == 1073)|\
                                 (store_cancel["store_id"] == 356)|(store_cancel["store_id"] == 528)]
store_cancel_ratio["store_id"] = ["161번", "1024번", "0번", "221번", "942번", "958번", "428번", "1073번", "356번", "528번"]

fig = go.Figure()
fig.add_trace(go.Bar(x=store_cancel_ratio["store_id"], y=store_cancel_ratio["cancel_ratio"],\
                     marker_color="#316395"))
fig.update_layout(title='거래 취소율 상위 10개 상점',
                  xaxis_title='상점 ID',
                  yaxis_title='거래 취소율(%)',
                  font_size=12)
fig.show()
```  
<img src="https://user-images.githubusercontent.com/72811950/105214233-14aaf380-5b93-11eb-9c44-9831529995be.png" width="700" height="500"></img>
- 거래취소가 많았던 161번 상점과 1024번 상점의 취소비율 또한 높은 것을 알 수 있었음.
- 161상점은 취소비율이 17.9%인 것을 알 수 있었다.  
  
## 4. 결론  
#### 1) 시간에 따른 탐색
- 시간의 흐름에 따라 2년 동안 매출은 상승하는 모습을 보인다.  
  이는 한 번에 결제되는 금액이 증가해서가 아니라 결제 횟수가 증가했기 때문이었다.
- 시즌별 : 겨울에는 매출이 낮았다가 봄이 되면서 매출이 연말까지 상승하는 모습을 보인다.
- 요일별 : 주말 전날인 금요일과 토요일에 매출이 높았다.
- 시간대별 : 점심과 저녁시간에 매출이 급상승한다.
#### 2) 할부 내역 탐색
- 가장 빈도수가 높았던 할부 월수는 3개월이었다.
- 요일별 : 토요일과 금요일에 할부 결제가 많이 일어난다.
- 시간대별 : 어느 시간대이든 할부 매출액이 일시불보다 크게 상회한다. 
#### 3) 거래 취소 내역 탐색
- 거래가 활발히 일어나는 12시와 19시에 거래 취소도 많이 일어난다.
- 거래 취소율은 어느 시간대이든 비슷하다.

## 5. 후기
#### 1) 아쉬웠던 점
- 처음 하게 된 프로젝트였다.  
  그래서 복잡한 데이터를 선택하는 것보다는 비교적 변수의 개수가 적은 데이터를 선택하는 것이 좋다고 생각했다.  
  만약 변수 개수가 더 많은 데이터를 선택했다면 다양한 인사이트를 도출하고 시각화했을 때 더 많은 그래프를 볼 수 있었을 것이다.  
  좀 더 데이터를 시각화하는데 익숙해지면 신용카드 사용지역과 상점의 유형이 추가된 데이터를 시각화할 계획이다.  
#### 2) 배운 점
- 데이터를 그룹화하고 그것을 바탕으로 그래프를 그리는 방법을 배웠다.  
  아직은 미숙하지만 시각화 라이브러리를 이용해 다양한 그래프를 그리는 법을 배우게 되어 앞으로 잘 이용할 수 있을 것 같다. 

