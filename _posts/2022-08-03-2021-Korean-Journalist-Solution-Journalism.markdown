---
layout: post
title: 2021 한국의 언론인 - 솔루션 저널리즘
image: 5.jpg
date: 2022-08-03 18:11:18 +0200
tags: [Project, 2021 Korean Journalist]
categories: Project
---
한국언론진흥재단은 매년 언론인을 대상으로 통계조사를 진행하고 있다. 한국언론진흥재단의 [언론인 조사]는 국내 유일의 언론인 대상 통계 조사다.

이번에는 [2021 한국의 언론인] 데이터에서 언론인 직업에 대한 만족도, 언론의 역할에 관한 문항에 집중하였다. 속해있는 언론사의 역할 수행이 기자의 직업 만족도와 효능감에 영향을 미칠 것이라 예상하였다. 언론은 대중에게 정확한 정보를 제공하고 사회 현안에 대한 다양한 의견을 전달하는 역할을 맡고 있다. 각자 속한 언론사가 이 역할을 잘 수행한다고 생각할 수록 기자의 조직에 대한 만족도와 직무의 효능감이 높아질 것이라 예상하였다.
언론의 역할 중 사회 현안에 대한 정확한 정보 제공, 다양한 의견 제시, 해결책 제시를 솔루션 저널리즘이라 정의하였다.

***
[2021 한국의 언론인 - 한국언론진흥재단](https://www.kpf.or.kr/front/research/reporterDetail.do?miv_pageNo=&miv_pageSize=&total_cnt=&LISTOP=&mode=W&seq=592375&link_g_topmenu_id=&link_g_submenu_id=&link_g_homepage=F&reg_stadt=&reg_enddt=&searchkey=all1&searchtxt=)
{% highlight markdown %}
* 조사 대상: 언론산업 종사자 중 기자직 종사자 2,014명
* 조사 방법: 대면 면접조사, 온라인/모바일 조사 병행
* 조사 기간: 2021년 7월 19일 ~ 10월 5일
자세한 내용은 홈페이지 참조
{% endhighlight %}

---

#### 데이터 불러오기

```python
import pandas as pd

#데이터 불러오기
df = pd.read_csv("2021_korean_journalist_new.csv")
```

지난 프로젝트인 [언론 내 성 불평등 문제 인식]과 같은 데이터를 사용한다. csv파일을 불러오기 위해서는 pandas 모듈을 불러온 후 pd.read_csv("")를 입력하면 된다.

---
#### 변수 생성 및 신뢰도 측정

![Alt text](/images/q2.jpg)

솔루션 저널리즘이 얼마나 중요한지 기자는 각 항목을 5점 척도로 응답하였다. 다음 문항에서는 실제로 속해있는 언론사가 그 역할을 어느 정도 실행하고 있는지 똑같은 문항을 두고 응답하였다. 세부 문항 1~7번 중 솔루션 저널리즘에 해당하는 1~4번 문항을 사용하기로 하였다. 역할 기대와 역할 수행의 간극을 독립변수로 사용하기 위해 새롭게 변수를 생성하였다.

코드는 다음과 같다.

```python
import pingouin as pg #Cronbach's alpha

#solution journalism (기자 기대)
solution_expected = df[['Q2_1', 'Q2_2', 'Q2_3', 'Q2_4']] #(McIntyre & Lough, 2021)
pg.cronbach_alpha(data=solution_expected) #신뢰도 측정 (cronbach alpha) a = .80
df['solution_expected'] = df[['Q2_1', 'Q2_2', 'Q2_3', 'Q2_4']].mean(axis=1)
df['solution_expected'].mean() # 4.17

#solution journalism (언론 수행)
solution = df[['Q3_1', 'Q3_2', 'Q3_3', 'Q3_4']]
pg.cronbach_alpha(data=solution) #신뢰도 측정 (cronbach alpha) a = .84
df['solution'] = df[['Q3_1', 'Q3_2', 'Q3_3', 'Q3_4']].mean(axis=1)
df['solution'].mean() # 3.48


#언론사 실행과 언론인들의 기대치 간 차이
df['difference'] = abs(df['solution'] - df['solution_expected'])
```

세부 문항을 묶은 새로운 변수의 크론바흐 알파 계수를 확인하기 위하여 pingouin 모듈을 사용하였다. pg.cronbach_alpha(data=)를 입력하면 설문 문항의 크론바흐 알파 계수 값을 얻을 수 있다. 새로 만든 두 변수 모두 0.8 이상으로 일관성을 가지고 있다.

각 세부 항목의 응답을 더한 후 평균 값을 기자 기대, 언론 수행 변수로 생성하였다. 솔루션 저널리즘의 기자 기대는 평균 값이 4.17이며 언론 수행은 평균 3.48이다. 두 변수의 간극을 위해 언론 수행 변수에서 기자 기대 변수를 뺀 값의 절대값(abs)을 간극(between) 변수로 설정하였다. 


![Alt text](/images/q3.jpg)
![Alt text](/images/q4.jpg)

종속 변수로 사용할 조직에 대한 만족도와 직업 효능감은 위 그림과 같은 설문 문항으로 측정되었다.

```python
#org. satisfaction: 조직만족
satisfaction = df[['Q29_1', 'Q29_2', 'Q29_3']]
pg.cronbach_alpha(data=satisfaction) #신뢰도 측정 (cronbach alpha) a = .87
df['satisfaction'] = df[['Q29_1', 'Q29_2', 'Q29_3']].mean(axis=1)
df['satisfaction'].mean() # 3.46

#journalistic efficacy: 직업 효능감
efficacy = df[['Q31_1', 'Q31_2']]
pg.cronbach_alpha(data=efficacy) #신뢰도 측정 (cronbach alpha) a = .83
df['efficacy'] = df[['Q31_1', 'Q31_2']].mean(axis=1)
df['efficacy'].mean() # 3.44
```

조직 만족도와 직업 효능감 문항 모두 상관계수 0.8 이상의 값을 가졌다. 조직 만족도의 평균 값은 3.46이며, 직업 효능감의 평균 값은 3.44이다.

```python
#Data frame creation
df_new = df[['gender', 'age', 'years', 'education', 'income', 'job_status', 'marital_status', 'id_stance',
               'perceived_ssclass', 'newstype', 'location', 'position', 'solution', 'solution_expected',
                'difference', 'satisfaction', 'efficacy']]

df_new = pd.get_dummies(df_new) #더미 변수 포함 데이터 셑
df_new #데이터 확인 with dummy variable
```

이번 프로젝트에서 사용할 변수로 새로운 데이터프레임을 생성하였다. pd.get_dummies는 범주형 변수를 더미 변수로 변환해주는 명령어다. 인구통계학적 변인이 문자로 들어가 있을 경우, 더미 변수로 변환하여 통계분석을 실시하는 것이 좋다. 본 프로젝트에는 newstype이라는 변수가 문자로 이루어져 있어서 더미변수로 변환하는 과정을 거쳤다.

---
#### 다중공선성

다중회귀분석을 실시하기 전, 독립변수 간 상관관계가 있으면 안되기 때문에 다중공선성을 확인하였다. 코드는 다음과 같다.


```python
#VIF용 패키지 불러오기
from statsmodels.formula.api import ols
from statsmodels.stats.outliers_influence import variance_inflation_factor

#VIF check ; multicollinearity (다중공선성)
vif_model = ols('satisfaction ~ age + gender + years + education + income + '
          'job_status + id_stance + newstype_papers + newstype_TV + newstype_internet + '
          'newstype_newswire + position', data = df_new)

#predictors 이름 확인
vif_model.exog_names

#VIF 한번에 보기
pd.DataFrame({'column': column, 'VIF': variance_inflation_factor(vif_model.exog, i)}
             for i, column in enumerate(vif_model.exog_names)
             if column != 'Intercept')
```
![Alt text](/images/t3.jpg)

다중공선성을 확인하는 방법 중, VIF(분산팽창요인)을 활용하기로 하였다. VIF의 값이 5~10이 넘으면 다중공산성이 있다고 판단한다. 회귀분석에 쓰일 인구통계학적 변인간에 다중공선성이 없는 것을 확인하였다.

---

#### 다중회귀분석

```python
from scipy import stats

#correlation b/w two variables (주요 변수만 확인했음)
stats.pearsonr(df_new['solution_expected'], df_new['solution']) #r = .26
stats.pearsonr(df_new['solution_expected'], df_new['difference']) #r = .44
stats.pearsonr(df_new['solution'], df_new['difference']) #r = -.63
stats.pearsonr(df_new['difference'], df_new['satisfaction']) #r= -.29
stats.pearsonr(df_new['difference'], df_new['efficacy']) #r= -.16
stats.pearsonr(df_new['satisfaction'], df_new['efficacy']) #r= .39
```

주요 변인간의 상관관계를 먼저 살펴봤다. 솔루션 저널리즘의 기자 기대와 실제 수행은 정적인 관계를 보였다. 기대와 수행은 두 변인의 간극과 부적의 관계를 나타냈다. 솔루션 저널리즘의 기대와 수행 간극이 커질 수록 조직 만족도와 직무 효능감은 떨어지는 부적의 관계를 가지는 것으로 나타났다. 

```python
#org.satisfaction
lm1 = ols('satisfaction ~ age + gender + years + education + income + '
          'id_stance + job_status + newstype_papers + newstype_TV + newstype_internet + '
          'newstype_newswire + position + difference', data = df_new).fit()

lm1.summary()#솔루션 역할 수행 수준과 기대치의 간극이 넓을수록 조직 만족도가 낮아짐 (B = -.34, SE = .03, p < .001)
```

![Alt text](/images/t4.jpg)

인구통계학적 변인을 통제변인으로 둔 후, 다중회귀분석을 실시하였다. 솔루션 저널리즘의 기대와 수행의 간극이 클 수록 조직 만족도가 낮아지는 점을 확인할 수 있었다. 회귀모형의 R-squared 값은 .124이다.

```python
#journalistic efficacy
lm2 = ols('efficacy ~ age + gender + years + education + income + '
          'id_stance + job_status + newstype_papers + newstype_TV + newstype_internet + '
          'newstype_newswire + position + difference', data = df_new).fit()

lm2.summary()#솔루션 역할 수행 수준과 기대치의 간극이 넓을수록 직무 효능감이 낮아짐 (B = -.16, SE = .02, p < .001)
```

![Alt text](/images/t5.jpg)

솔루션 저널리즘의 기대와 수행 간극이 커질 수록 직무 효능감이 낮아지는 것을 확인하였다. 조직 만족도에 비해 회귀모형의 설명력이 낮아지고, 독립변수(간극)의 B값도 낮아진 점을 볼 수 있다.


```python
#IV = difference, DV = satisfaction conditional upon gender
lm3 = ols('satisfaction ~ age + years + education + income + '
          'id_stance + job_status + newstype_papers + newstype_TV + newstype_internet + '
          'newstype_newswire + position + difference*gender', data = df_new).fit()

lm3.summary() # 위 직접 효과 관계 (difference와 만족도)는 성별에 따라 유의미한 차이가 존재함
```

![Alt text](/images/t6.jpg)

독립변수인 솔루션 저널리즘의 간극과 종속변수인 조직 만족도의 관계에 성별을 조절변수로 설정하여 분석을 실시하였다. 두 변인의 관계가 성별에 따라 유의미한 차이가 존재하는 것으로 나타났다.

```python
lm4 = ols('efficacy ~ age + years + education + income + '
          'id_stance + job_status + newstype_papers + newstype_TV + newstype_internet + '
          'newstype_newswire + position + difference*gender', data = df_new).fit()

lm4.summary() # 위 직접 효과 관계 (difference와 만족도)는 성별에 따라 유의미한 차이가 존재하지 않음
```

![Alt text](/images/t7.jpg)

앞서 본 솔루션 저널리즘의 간극과 종속변수인 직무 효능감은 직접적으로 유의미한 관계가 있었으나, 성별에 따라서는 유의미한 차이가 존재하지 않았다.

```python
import seaborn as sns

sns.lmplot(x = 'difference', y = 'satisfaction', data = df_new, hue = 'gender', palette="Set2", scatter_kws={'alpha':0.5})
```

유의미한 차이를 보였던 솔루션 저널리즘의 간극과 조직 만족도 관계와 성별을 그래프로 표현하였다.

![Alt text](/images/g7.jpg)

솔루션 저널리즘의 기대와 실제 수행의 간극이 커질 수록 조직 만족도가 떨어지는 양상을 확인할 수 있으며, 남성(=0)보다 여성(=1)이 기울기가 큰 것으로 나타났다.

---

이번 프로젝트는 솔루션 저널리즘에 대한 기대와 실제 수행이 기자의 조직 만족도와 직무 효능감에 어떠한 영향을 미치는 지에 대해 살펴봤다. 분석을 실시한 결과, 솔루션 저널리즘에 대한 기대와 실제 수행의 간극이 조직 만족도에 유의미한 영향을 미치는 것으로 나타났다. 솔루션 저널리즘에 대한 기자의 기대와 언론사의 실제 수행의 차이가 클수록 기자의 조직 만족도가 낮아지는 것이다. 즉 언론인들이 솔루션 저널리즘의 중요성을 알고 있으며, 자신이 속한 언론사가 솔루션 저널리즘을 잘 수행하지 못한다고 생각하면 조직에 대한 만족도가 떨어진다는 것이다. 이러한 결과는 앞으로 언론의 보도가 단순 정보 제공이 아닌 다양한 의견과 해결책을 제시하는 솔루션 저널리즘을 추구해야만 기자들도 조직에 대해 만족할 수 있다는 점을 시사한다.
다른 종속 변인이었던 직무 효능감은 솔루션 저널리즘의 간극과 직접 효과 관계는 유의미했지만, 성별의 조절 효과는 유의미한 차이를 보이지 않았다.

SPSS를 활용하여 회귀분석을 수행한 경험은 많지만, Python을 활용한 것은 이번이 처음이다. 인구통계학적 변인을 통제변인으로 둔 다중회귀분석을 실시하였고, 성별의 조절효과도 확인하였다. 직책에 따른 조절효과를 확인하였지만 예측과는 다르게 유의미한 차이를 보이지 않았다. 이번 프로젝트를 진행하며 Python을 활용한 통계분석이 한층 더 익숙해졌다.

---

#### 전체 Code
[2021 한국의 언론인 - 솔루션 저널리즘] 프로젝트의 전체 코드는 다음과 같다.

```python
import pandas as pd
import pingouin as pg #Cronbach's alpha
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

#solution journalism (기자 기대)
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
