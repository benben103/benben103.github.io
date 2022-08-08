---
layout: post
title: 2021 Korean Journalist - Solution Journalism
image: 5.jpg
date: 2022-08-03 18:11:18 +0200
tags: [Project, 2021 Korean Journalist]
categories: Project
---
한국언론진흥재단은 매년 언론인을 대상으로 통계조사를 진행하고 있다. 한국언론진흥재단의 [언론인 조사]는 국내 유일의 언론인 대상 통계 조사다.

본 연구자는 [2021 한국의 언론인] 데이터에서 속해있는 조직과 언론인이라는 직업에 대한 만족도, 언론의 역할에 관한 문항에 집중하였다.

***
[2021 한국의 언론인 - 한국언론진흥재단](https://www.kpf.or.kr/front/research/reporterDetail.do?miv_pageNo=&miv_pageSize=&total_cnt=&LISTOP=&mode=W&seq=592375&link_g_topmenu_id=&link_g_submenu_id=&link_g_homepage=F&reg_stadt=&reg_enddt=&searchkey=all1&searchtxt=)
{% highlight markdown %}
* 조사 대상: 언론산업 종사자 중 기자직 종사자 2,014명
* 조사 방법: 대면 면접조사, 온라인/모바일 조사 병행
* 조사 기간: 2021년 7월 19일 ~ 10월 5일
자세한 내용은 홈페이지 참조
{% endhighlight %}

#### Code
```python
import pandas as pd
from numpy import mean
import matplotlib.pyplot as plt
import seaborn as sns
from statsmodels.stats.multicomp import pairwise_tukeyhsd
from statsmodels.formula.api import ols
from statsmodels.stats.anova import anova_lm
from scipy import stats
from mpl_toolkits.mplot3d import Axes3D #3D chart

#데이터 불러오기
df = pd.read_csv("2021_korean_journalist_new.csv")
df#데이터 확인

#Variable Creation

#언론사구분
df['newstype'].replace({1:'papers',2:'TV',3:'internet',4:'telecom'}, inplace=True)
df['working_type'].replace({1:'1',2:'0'}, inplace=True) #1 = 정규직 0 = 비정규직
df['position'] = df['position'].replace({1:5, 2:4, 3:3, 4:2, 5:1})

#solution journalism (기자 인식)
solution_expected = df[['Q2_1', 'Q2_2', 'Q2_3', 'Q2_4']] #(McIntyre & Lough, 2021)
pg.cronbach_alpha(data=solution_expected) #신뢰도 측정 (cronbach alpha) a = .80
df['solution_expected'] = df[['Q2_1', 'Q2_2', 'Q2_3', 'Q2_4']].mean(axis=1)

#solution journalism (언론 수행)
solution = df[['Q3_1', 'Q3_2', 'Q3_3', 'Q3_4']]
pg.cronbach_alpha(data=solution) #신뢰도 측정 (cronbach alpha) a = .84
df['solution'] = df[['Q3_1', 'Q3_2', 'Q3_3', 'Q3_4']].mean(axis=1)

#언론사 실행과 언론인들의 기대치 간 차이
df['difference'] = abs(df['solution'] - df['solution_expected'])

#org. satisfaction: 조직만족
satisfaction = df[['Q29_1', 'Q29_2', 'Q29_3']]
pg.cronbach_alpha(data=satisfaction) #신뢰도 측정 (cronbach alpha) a = .87
df['satisfaction'] = df[['Q29_1', 'Q29_2', 'Q29_3']].mean(axis=1)

#journalistic efficacy: 직업 효능감
efficacy = df[['Q31_1', 'Q31_2']]
pg.cronbach_alpha(data=efficacy) #신뢰도 측정 (cronbach alpha) a = .83
df['efficacy'] = df[['Q31_1', 'Q31_2']].mean(axis=1)

#Data frame creation
df_new = df[['gender', 'age', 'years', 'education', 'income', 'job_status', 'marital_status', 'id_stance',
               'perceived_ssclass', 'newstype', 'location', 'position', 'solution', 'solution_expected',
                'difference', 'satisfaction', 'efficacy']]

df_new = pd.get_dummies(df_new) #더미 변수 포함 데이터 셑
df_new #데이터 확인 with dummy variable

#Descriptive Statistics: 기술 통계 확인
print(df_new['age'].describe()) #m = 40.57, SD = 9.99
df_new['gender'].value_counts() #m = 1370, female = 644
print(df_new['years'].describe()) #m = 12.88, SD = 8.98
print(df_new['education'].describe()) #m = 3.38, SD = .96
print(df_new['income'].describe()) #m = 4.80, SD = 2.38
df_new['marital_status'].value_counts() #결혼 = 1139, 비결혼 = 875
df_new['job_status'].value_counts() #정규직 = 1909, 비정규직 = 105
print(df_new['id_stance'].describe()) #m = -.34, SD = 1.75
print(df_new['perceived_ssclass'].describe()) #m = 5.03, SD = 1.40
df['newstype'].value_counts() #신문 = 1003, 인터넷언론 = 491, 방송 = 384, 뉴스통신 = 136; 더미 변수
df['position'].value_counts() #평기자 = 1090, 차장 = 414, 부장 = 213, 국장 = 149, 부국장 = 118
print(df_new['solution'].describe()) #m = 3.48, SD = .68
print(df_new['solution_expected'].describe()) #m = 4.17, SD = .62
print(df_new['difference'].describe()) #m = .80, SD = .68
print(df_new['satisfaction'].describe()) #m = 3.46, SD = .85
print(df_new['efficacy'].describe()) #m = 3.44, SD = .75

#VIF check ; multicollinearity (다중공선성)
vif_model = ols('satisfaction ~ age + gender + years + education + income + '
          'job_status + id_stance + newstype_papers + newstype_TV + newstype_internet + '
          'newstype_newswire + position', data = df_new)

#VIF용 패키지 불러오기
from statsmodels.stats.outliers_influence import variance_inflation_factor

#predictors 이름 확인
vif_model.exog_names

#VIF 한번에 보기
pd.DataFrame({'column': column, 'VIF': variance_inflation_factor(vif_model.exog, i)}
             for i, column in enumerate(vif_model.exog_names)
             if column != 'Intercept') #VIF 값이 5이하: predictors의 독립성이 보장됨 --> good to go!

#변수 histogram 확인
df_new['solution'].hist()
df_new['solution_expected'].hist()
df_new['difference'].hist()
df_new['satisfaction'].hist()
df_new['efficacy'].hist()

#correlation
Matrix = df_new.corr(method='pearson', min_periods = 1).round(2)
print(Matrix)

#correlation b/w two variables (주요 변수만 확인했음)
stats.pearsonr(df_new['solution'], df_new['solution_expected']) #r = .26
stats.pearsonr(df_new['difference'], df_new['satisfaction']) #r= -.29
stats.pearsonr(df_new['difference'], df_new['efficacy']) #r= -.16
stats.pearsonr(df_new['satisfaction'], df_new['efficacy']) #r= .39

######################Direct-effects  relationship#################################

#org.satisfaction
lm1 = ols('satisfaction ~ age + gender + years + education + income + '
          'id_stance + job_status + newstype_papers + newstype_TV + newstype_internet + '
          'newstype_newswire + position + difference', data = df_new).fit()

lm1.summary()#솔루션 역할 수행 수준과 기대치의 간극이 넓을수록 조직 만족도가 낮아짐 (B = -.34, SE = .03, p < .001)

#journalistic efficacy
lm2 = ols('efficacy ~ age + gender + years + education + income + '
          'id_stance + job_status + newstype_papers + newstype_TV + newstype_internet + '
          'newstype_newswire + position + difference', data = df_new).fit()

lm2.summary()#솔루션 역할 수행 수준과 기대치의 간극이 넓을수록 직무 효능감이 낮아짐 (B = -.16, SE = .02, p < .001)

#Centering the predictor for Interaction Term
center_function = lambda x: x - x.mean()
df_new['difference'] = center_function(df_new['difference'])

#charts

#sns.lmplot(x = 'difference', y = 'satisfaction', data = df_new, ci=10, line_kws={"color":"r", "alpha":1.0})
#g.set(xlim=(0, 10))
#g.set(ylim=(0, 10))

g = sns.lmplot(x = 'difference', y = 'satisfaction', data = df_new, ci=50, scatter_kws={"s":150, "alpha":0.1})
g.set(xlim=(10, 100))
g.set(ylim=(0, 50))

#######################interaction term conditional upon gender###################################

#IV = difference, DV = satisfaction conditional upon gender
lm3 = ols('satisfaction ~ age + years + education + income + '
          'id_stance + job_status + newstype_papers + newstype_TV + newstype_internet + '
          'newstype_newswire + position + difference*gender', data = df_new).fit()

lm3.summary() # 위 직접 효과 관계 (difference와 만족도)는 성별에 따라 유의미한 차이가 존재함

#IV = difference, DV = satisfaction conditional upon position
lm4 = ols('efficacy ~ age + years + education + income + '
          'id_stance + job_status + newstype_papers + newstype_TV + newstype_internet + '
          'newstype_newswire + position + difference*gender', data = df_new).fit()

lm4.summary() # 위 직접 효과 관계 (difference와 만족도)는 성별에 따라 유의미한 차이가 존재하지 않음

#Interaction Term: 시각화 (필요하면 사용 가능)
sns.lmplot(x = 'difference', y = 'satisfaction', data = df_new, hue = 'gender', palette="Set1", scatter_kws={'alpha':0.5})
sns.lmplot(x = 'difference', y = 'satisfaction', data = df_new, col = 'gender')
sns.lmplot(x = 'difference', y = 'efficacy', data = df_new, hue = 'gender', palette="Set1", scatter_kws={'alpha':0.5})
sns.lmplot(x = 'difference', y = 'efficacy', data = df_new, col = 'gender')
```
