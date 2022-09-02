---
layout: post
title: 2021 한국의 언론인 - 언론 내 성 불평등 인식
image: 13.jpg
date: 2022-08-03 18:23:20 +0200
tags: [Project, 2021 Korean Journalist]
categories: Project
---
한국언론진흥재단은 매년 언론인을 대상으로 통계조사를 진행하고 있다. [언론인 조사]는 국내 유일의 언론인 대상 통계 조사다. 

[2021 한국의 언론인] 데이터에서 언론사의 성 불평등 인식을 집중하여 살펴보았다. 최근 몇 년간 젠더 갈등이 큰 이슈로 대두되었다. 많은 사람들이 관심을 가지고 있는 주제인 만큼 언론 역시 집중해서 보도하는 경향을 보였다. 이처럼 갈등이 심화되고 있는 상황에서 언론은 중립적인 입장을 취하는 것이 중요하다. 따라서 현재 언론 내에서의 성 불평등 인식을 중점으로 살펴보았다.

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

df = pd.read_csv("2021_gender.csv")
df_gender = df[['gender', 'age', 'years', 'education', 'income', 'job_status', 'marital_status', 'id_stance',
               'children', 'perceived_ssclass', 'newstype', 'news_form', 'location', 'position', 'gender_equity', 'gender_perception']]

#female dataframe 생성
female = (df_gender.gender == 1)
df_female = df_gender.loc[female, ['gender', 'age', 'years', 'education', 'income', 'job_status', 'marital_status', 'id_stance',
               'children', 'perceived_ssclass', 'newstype', 'news_form', 'location', 'position', 'gender_equity', 'gender_perception']]
df_female

#male dataframe 생성
male = (df_gender.gender == 0)
df_male = df_gender.loc[male, ['gender', 'age', 'years', 'education', 'income', 'job_status', 'marital_status', 'id_stance',
               'children', 'perceived_ssclass', 'newstype', 'news_form', 'location', 'position', 'gender_equity', 'gender_perception']]
df_male
```

기존에 존재하고 있는 csv 파일을 불러오기 위해서 pandas 플러그인을 사용하였다. 분석에 쓰여질 변인(ex. 성별, 나이, 직위, 연차, 성 불평등 문제 인식 등)으로 이루어진 데이터프레임을 생성하였다.
성별에 따른 분석을 보다 편하게 하기 위하여 "(df_gender.gender == 1),(df_gender.gender == 0)" 이라는 조건을 달아 남성과 여성의 데이터프레임을 구분지어 생성해두었다.

---

#### 상관관계 분석
![Alt text](/images/q1.jpg)

성 불평등 문제 인식은 5점 척도로, 총 4개의 문항으로 이루어져 있다. 숫자가 5에 가까울 수록 성 불평등 문제를 심각하게 받아들인다 할 수 있다. 가장 먼저 나이 (연차)와 성 불평등 문제 인식이 어떤 관계가 있는 지 살펴보았다.
피어슨 상관계수를 구하기 위해 scipy의 stats 플러그인을 사용하였다. 사용법은 다음과 같다.
```python
from scipy import stats

test = stats.pearsonr(x,y)

print(test)
```

```python 
from scipy import stats

#상관관계: 나이(연차)와 성 불평등 문제 인식
age_gender_cor = stats.pearsonr(df_gender['age'], df_gender['gender_perception'])
print("%.2f, %.2f"% age_gender_cor) #-0.214, 0.000

years_gender_cor = stats.pearsonr(df_gender['years'], df_gender['gender_perception'])
print("%.2f, %.2f"%years_gender_cor) #-0.137, 0.000
```

나이와 성 불평등 문제 인식의 상관계수는 -0.214로 부적인 관계를 보였다. 즉 나이가 많을 수록 성 불평등 문제를 심각하게 받아들이지 않는다고 할 수 있다. 연차 역시 부적인 관계를 보였다.

```python 
import matplotlib.pyplot as plt
import seaborn as sns

#나이 lineplot
sns.set_style('darkgrid')
plt.rcParams["font.family"] = 'Yoon YGO 550_TT'
fig, ax = plt.subplots(figsize=(8,6))
sns.lineplot(x='age', y='gender_perception', ci=None, data=df_gender, ax=ax)
ax.set_title('나이와 성 불평등 문제 인식', x=0.5, y=1.05, fontsize=14)
plt.xlabel('나이')
plt.ylabel('성 불평등 문제 인식')
plt.show()
```
![Alt text](/images/g1.jpg)

나이와 성 불평등 문제 인식의 관계를 선 그래프로 나타냈다. 그래프를 그리기 위하여 matplot과 seaborn 플러그인을 사용하였다. 그래프의 제목, 레이블 추가와 사이즈, 색상 변경 등에 관한 코드 내용은 추후에 정리할 예정이다.

---

#### ANOVA 분석

나이와 성 불평등 문제 인식의 관계를 더 보기 위하여 세대별로 범주화하였다. 보다 편리하게 세대별 범주화를 하기 위하여 birth(태생 연도) 변수를 새로 생성하였다. 밑 링크를 참조하여 베이비 부머, X, Y, Z세대로 범주화 하였다.

```python 
#나이 세대별 범주화 (참조: https://dailyfreepress.com/2021/03/15/generation-names-explained/)
df_gender['birth'] = 2022 - df_gender['age']
df_gender.loc[df_gender['birth'] > 1997, 'birth'] = 4
df_gender.loc[df_gender['birth'] > 1981, 'birth'] = 3
df_gender.loc[df_gender['birth'] > 1965, 'birth'] = 2
df_gender.loc[df_gender['birth'] > 1946, 'birth'] = 1
df_gender['generation'] = df_gender['birth'].replace({1:'Baby Boomer', 2:'Generation X', 3:'Generation Y', 4:'Generation Z'})
```

세대간 성 불평등 문제 인식에 유의미한 차이가 있는 지 확인하기 위하여 ANOVA(분산분석)을 실시하였다. 사후분석은 tukey 검정을 사용하였다.
```python 
from statsmodels.formula.api import ols
from statsmodels.stats.anova import anova_lm
from statsmodels.stats.multicomp import pairwise_tukeyhsd

#ANOVA
model = ols('종속변수 ~ C(독립변수)', 데이터프레임).fit()
print(anova_lm(model))

#Tukey
hsd = pairwise_tukeyhsd(데이터프레임['종속변수'], 데이터프레임['독립변수'], alpha=0.05)
print(hsd.summary())
```

```python 
from statsmodels.formula.api import ols
from statsmodels.stats.anova import anova_lm
from statsmodels.stats.multicomp import pairwise_tukeyhsd

#세대별 성 불평등 문제 인식 ANOVA, 사후분석
model = ols('gender_perception ~ C(generation)', df_gender).fit()
print(anova_lm(model)) #F=24.07 P < 0.05
hsd = pairwise_tukeyhsd(df_gender['gender_perception'], df_gender['generation'], alpha=0.05)
print(hsd.summary()) #BB - Y, BB - Z, X -Y
```
![Alt text](/images/t1.jpg)

세대간 성 불평등 문제 인식을 살펴본 결과, 유의미한 차이가 있는 것으로 확인되었다. 사후 검정을 통해 어떤 집단이 서로 차이가 있는지 확인하였다. 베이비 부머 세대가 Y, Z세대와 유의미한 차이가 있었으며, X세대와 Y세대가 통계적으로 유의미한 차이를 보였다.

```python 
#세대별 boxplot
sns.set_style('darkgrid')
plt.rcParams["font.family"] = 'Yoon YGO 550_TT'
fig, ax = plt.subplots(figsize=(8,6))
sns.boxplot(x='generation', y='gender_perception', order=['Baby Boomer\n(1946-1964)', 'Generation X\n(1965-1980)',
                                                          'Generation Y\n(1981-1996)', 'Generation Z\n(1997-2012)'], data=df_gender, ax=ax)
ax.set_title('세대별 성 불평등 문제 인식', x=0.5, y=1.05, fontsize=14)
#ax.text(x=1.0, y=1.01, s='F=-24.07, p<0.05', fontsize=10, alpha=0.75, ha='right', va='bottom', transform=ax.transAxes)
plt.xlabel('세대 구분')
plt.ylabel('성 불평등 문제 인식')
plt.show()
```
![Alt text](/images/g2.jpg)

세대간 성 불평등 문제 인식 관계를 박스 그래프로 나타냈다. 그래프에서 보이는 것처럼 앞 세대가 비교적 성 불평등 문제 인식이 낮을 점을 확인할 수 있다.

---

#### t-test 분석

성평등과 관련한 프로젝트로 종속변수인 성 불평등 문제 인식에 가장 큰 영향을 주는 변수는 성별일 것이라 예상하였다. 성별간 성 불평등 문제 인식이 어떻게 나타나는지 보기 위하여 t-test 분석을 실시하였다.

```python 
#집단 구분 (남성/여성)
perception_male = df_gender.loc[df_gender['gender']=='남성', 'gender_perception']
perception_female = df_gender.loc[df_gender['gender']=='여성', 'gender_perception']

levene =stats.levene(perception_male, perception_female)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.633

ttest = stats.ttest_ind(perception_male, perception_female, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = -18.842 p = 0.000

#성별에 따른 성 불평등 문제 인식 barplot
sns.set_style('darkgrid')
colors = ["#95B7F0", "#F07674"]
sns.set_palette(sns.color_palette(colors))
plt.rcParams["font.family"] = 'Yoon YGO 550_TT'
fig, ax = plt.subplots(figsize=(8,6))
sns.barplot(x='gender', y='gender_perception', order=['남성','여성'], data=df_gender, ax=ax)
ax.set_title('성별간 성 불평등 문제 인식', x=0.5, y=1.05, fontsize=14)
#ax.text(x=1.0, y=1.01, s='t=-18.84, p<0.05', fontsize=10, alpha=0.75, ha='right', va='bottom', transform=ax.transAxes)
ax.annotate(text='평균=2.41', xy=(0, 1), xycoords='data', ha='center', fontsize=10, alpha=1)
ax.annotate(text='평균=3.11', xy=(1, 1), xycoords='data', ha='center', fontsize=10, alpha=1)
plt.xlabel('성별')
plt.ylabel('성 불평등 문제 인식')
plt.show()
```
![Alt text](/images/g3.jpg)

분석 결과, 남성과 여성간 성 불평등 문제 인식에 유의미한 차이가 있는 것을 밝혀냈다. 남성의 성 불평등 문제 인식 평균은 5점 중 2.41인 것에 비해, 여성은 3.11로 남성보다 성 불평등 문제를 심각하게 받아들인다고 할 수 있다.

---

#### 교차분석

앞선 분석을 통해서 나이와 직책, 성별이 성 불평등 인식과 밀접한 관계가 있다고 판단하여 성별과 직책을 중점으로 더 살펴보기로 하였다.
직위와 성별의 교차표를 생성 후에, 카이스퀘어 검정을 실시하였다.

```python 
#직위와 성별 crosstab & chi-square
df_gender['position_r'] = df_gender['position'].replace({'국장':5, '부국장':4, '부장':3, '차장':2, '평기자':1})
pd.crosstab(df_gender["gender"], df_gender["position_r"])
contingency = pd.crosstab(df_gender["gender"], df_gender["position_r"])
print(contingency)
chi, p , dof, expected = stats.chi2_contingency(contingency)
print(f"카이스퀘어 검정통계량: {chi:.2f}",
      f"p-value(0.05): {p:.2f}",
      f"자유도: {dof}", sep = "\n" ) #184.854, 0.000, 4
```

pandas를 이용하면 교차표를 만들 수 있으며, stats의 chi2_contingency 모듈을 사용하면 교차표의 카이스퀘어 검정을 실시할 수 있다. 

위 코드를 실행하면 다음과 같은 결과가 나타난다.

![Alt text](/images/t2.jpg)

카이스퀘어 검정 결과, 성별과 직위 간에는 유의한 연관성이 있는 것으로 밝혀졌다(1은 평기자, 5는 국장 순). 직위가 높아질 수록 남녀의 비율의 차이가 커지는 것을 볼 수 있다.

```python 
#직위에 따른 성별 비율 barplot
df_gender.rename(columns = {"gender":"성별"},inplace=True)
colors = ["#95B7F0", "#F07674"]
sns.set_palette(sns.color_palette(colors))
sns.set_style('darkgrid')
plt.rcParams["font.family"] = 'Yoon YGO 550_TT'
fig, ax = plt.subplots(figsize=(8,6))
plt.title('직위에 따른 성별 비율', x=0.5, y=1.05, fontsize=14)
sns.countplot(x='position', hue='성별', hue_order=['남성','여성'], data=df_gender, ax=ax)
ax.annotate(text='56%', xy=(-0.25, 620), xycoords='data', ha='left', fontsize=8, alpha=1)
ax.annotate(text='44%', xy=(0.15, 490), xycoords='data', ha='left', fontsize=8, alpha=1)
ax.annotate(text='74%', xy=(0.75, 316), xycoords='data', ha='left', fontsize=8, alpha=1)
ax.annotate(text='26%', xy=(1.15, 118), xycoords='data', ha='left', fontsize=8, alpha=1)
ax.annotate(text='86%', xy=(1.75, 220), xycoords='data', ha='left', fontsize=8, alpha=1)
ax.annotate(text='14%', xy=(2.15, 43), xycoords='data', ha='left', fontsize=8, alpha=1)
ax.annotate(text='89%', xy=(2.75, 115), xycoords='data', ha='left', fontsize=8, alpha=1)
ax.annotate(text='11%', xy=(3.15, 23), xycoords='data', ha='left', fontsize=8, alpha=1)
ax.annotate(text='93%', xy=(3.75, 149), xycoords='data', ha='left', fontsize=8, alpha=1)
ax.annotate(text='7%', xy=(4.15, 20), xycoords='data', ha='left', fontsize=8, alpha=1)
plt.xlabel('직위')
plt.show()
```

![Alt text](/images/g4.jpg)

직위에 따른 성별 비율 한 눈에 보기 위하여 막대 그래프로 나타내는 코드와 그 결과이다. 평균 값은 앞선 교차표를 토대로 계산하여 직접 입력하였다. 이를 통해 언론사 내에 구조적으로 남녀간 직책에 차이가 있는 점을 알 수 있었다.

---

#### 이후 분석

성 불평등 인식에 가장 큰 영향을 미치는 변수는 성별이라는 것을 앞선 분석을 통해 간접적으로 알 수 있었다. 따라서 유의미한 관계를 보였던 나이와 직책을 남,여로 나누어 분석을 진행해보기로 하였다.

```python
#남성인 경우, 나이와 성 불평등 문제 인식 간 상관관계
age_male_cor = stats.pearsonr(df_male['age'], df_male['gender_perception'])
print("%.2f, %.2f"% age_male_cor) # -0.13, 0.00

age_female_cor = stats.pearsonr(df_female['age'], df_female['gender_perception'])
print("%.2f, %.2f"% age_female_cor) # -0.06 0.03

#나이와 성 불평등 문제 인식 linplot (남성/여성)
sns.set_style('darkgrid')
plt.rcParams["font.family"] = 'Yoon YGO 550_TT'
colors = ["#95B7F0", "#F07674"]
sns.set_palette(sns.color_palette(colors))
fig, ax = plt.subplots(figsize=(8,6))
sns.lineplot(x='age', y='gender_perception', label='남성인 경우', ci=None, data=df_male)
sns.lineplot(x='age', y='gender_perception', label='여성인 경우', ci=None, data=df_female)
ax.set_title('나이에 따른 성 불평등 문제 인식 (남성인 경우 / 여성인 경우)', x=0.5, y=1.05, fontsize=14)
plt.xlabel('나이')
plt.ylabel('성 불평등 문제 인식')
plt.legend
plt.show()
```

![Alt text](/images/g5.jpg)

먼저 나이에 따른 상관관계를 살펴봤다. 남녀가 같이 있는 데이터 셋에서는 상관계수 -0.214 (p<0.01)로 유의미한 관계를 보였다. 남,여를 나누어 상관분석을 실시한 결과는 남성인 경우 -0.13 (p>0.01), 여성의 경우는 -0.06 (p>0.05)로 나타났다.
두 그룹 모두 부적인 상관관계를 보이고 있지만 여성이 성 불평등 문제 인식이 비교적 남성보다 높은 것을 그래프를 통해 확인할 수 있었다.

다음으로는 직책에 따른 성 불평등 문제 인식을 남,여로 나누어 분석을 진행하였다. 두 집단을 비교하고자 5개의 척도로 나누어진 직책 변수를 2개의 집단으로 구성하였다(간부: 국장, 부국장, 부장, 차장 / 사원: 평기자)

```python
#position 집단 구분 (국장,부국장,부장,차장/평기자)
df_gender['position_group'] = df_gender['position'].replace({'국장':'간부','부국장':'간부','부장':'간부','차장':'간부','평기자':'사원'})

df_female['position_group'] = df_female['position'].replace({'국장':'간부','부국장':'간부','부장':'간부','차장':'간부','평기자':'사원'})
df_female['position_group'].value_counts()

df_male['position_group'] = df_male['position'].replace({'국장':'간부','부국장':'간부','부장':'간부','차장':'간부','평기자':'사원'})
df_male['position_group'].value_counts()

female_group_a = df_female.loc[df_female['position_group']=='간부', 'gender_perception']
female_group_b = df_female.loc[df_female['position_group']=='사원', 'gender_perception']

male_group_a = df_male.loc[df_male['position_group']=='간부', 'gender_perception']
male_group_b = df_male.loc[df_male['position_group']=='사원', 'gender_perception']

#간부와 사원 간 성 불평등 t-test
levene =stats.levene(female_group_a, female_group_b)
print("levenresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정
ttest = stats.ttest_ind(female_group_a, female_group_b, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t-value = -2.21, p-value = 0.03

levene =stats.levene(male_group_a, male_group_b)
print("levenresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정
ttest = stats.ttest_ind(male_group_a, male_group_b, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t-value = -2.39, p-value = 0.02

df_gender.rename(columns = {"gender":"성별"},inplace=True)

sns.set_style('darkgrid')
colors = ["#95B7F0", "#F07B71"]
sns.set_palette(sns.color_palette(colors))
plt.rcParams["font.family"] = 'Yoon YGO 550_TT'
fig, ax = plt.subplots(figsize=(8,6))
sns.boxplot(x='position_group', y='gender_perception', hue="성별", hue_order=['남성','여성'], data=df_gender)
ax.set_title('직위 그룹별 성 불평등 문제점 인식 (남/녀 구분)', x=0.5, y=1.05, fontsize=14)
#ax.text(x=1.0, y=1.01, s='t= -2.21, p<0.05', fontsize=10, alpha=0.75, ha='right', va='bottom', transform=ax.transAxes)
plt.xlabel('직위 그룹 구분')
plt.ylabel('성 불평등 문제 인식')
plt.show()
```

이 결과 역시 남,여를 나누어도 두 그룹(간부/사원)간 유의미한 차이가 있다고 나타났다. 즉 성별이 성 불평등 문제 인식에 큰 영향을 주지만 나이와 직책 또한 중요한 변수라는 점을 확인할 수 있다.

위의 코드를 진행하면 다음과 같은 그래프를 확인할 수 있다.

![Alt text](/images/g6.jpg)

---

#### 마무리
[언론 내 성 불평등 문제 인식]은 SPSS가 아닌 Python을 활용하여 분석을 실시한 첫 번째 프로젝트이다. 한국의 언론인 조사를 살펴 보던 중 성 불평등 문제 인식에 관련한 설문항목이 눈에 들어왔다. 최근 몇 년 동안 남녀 갈등이 중요한 이슈로 대두되고 있다. 특히 남녀 갈등 이슈는 20,30대를 중심으로 하여 이는 선거 결과에도 잘 나타났다. 20, 30대의 투표 결과를 보면 서로 다른 정당을 지지하는 것을 확인할 수 있었다. 
남녀 갈등은 언론도 집중하고 있는 문제이기도 하다. 이 프로젝트가 언론인의 성 불평등 문제 인식과 언론사 내 구조를 확인하는 동시에, 통계 분석을 연습할 수 있는 좋은 기회라고 생각하였다.

이번 프로젝트에 결과를 보면, 언론인의 나이가 많을 수록, 직책이 높을 수록 성 불평등 문제 인식은 낮게 나타나는 점을 밝혔다. 이는 남녀 갈등이 20,30대를 중심으로 나타나는 것과 비슷한 맥락이라고 판단된다. 예상했던 바와 같이 남성보다 여성의 성 불평등 문제 인식이 높은 점을 실제로 확인할 수 있었다. 흥미로운 점은 직책에 따른 성별 비율이다. 회사 내 구조적 문제는 성 불평등 주제에 자주 등장하는 이슈이다. '유리천장'이라는 개념이 있을 만큼 높은 직책에 남성의 비율이 높다는 점을 많은 전문가들이 지적해왔다. 이번 연구를 통해 언론사 또한 높은 직책일 수록 남녀의 비율에 큰 차이를 보이는 것을 확인할 수 있었다.
남성 / 여성을 나눈 후에도 나이와, 직책이 올라갈 수록 성 불평등 문제 인식이 낮게 나타난 결과도 흥미롭다. 남성의 성 불평등 문제 인식이 여성에 비해 낮은 것은 사실이지만, 나이와 직책에 따른 차이가 유의미하다는 점을 발견하였다. 즉 남성, 여성의 성 불평등 문제 인식에 유의미한 차이가 있으며, 직책과 나이는 성 불평등 문제 인식과 부적인 관계를 나타낸다는 점을 이번 프로젝트에서 확인할 수 있었다.

코드를 짜면서는 가장 먼저, 위 글에 포함되지 않았지만 csv 파일을 불러와 작업하기 쉽게 변수 이름을 변경하는 작업을 거쳤다. 이번 프로젝트에 필요한 변수는 몇 안되지만 연습을 한다는 생각으로 사용할 변수가 아닌 다른 변수들도 모두 변경하는 작업을 거쳤다. 전체 항목이 70개가 넘었기에 꽤 오랜 시간이 걸렸다. 변수 정리를 마친 후, 본격적으로 분석을 실시하였다. 이번 프로젝트에서는 교차분석, 상관분석, t-test, ANOVA를 수행하였다. Python으로 이러한 통계분석을 해본 적이 없었기에 검색을 통해 코드를 알 수 있었다.
이번 프로젝트에서 사용한 통계분석 코드는 이후 진행한 프로젝트에서도 잘 사용하고 있다. 통계분석과 시각화 코드에 대한 자세한 설명은 추후에 게시글을 작성할 예정이다. 아래 전체 코드에는 본문에 등장하지 않았던 연차, 결혼 여부 변수도 포함되어 있으니 실행해보는 것도 좋은 선택이라고 생각한다.

---
#### 전체 Code
```python 
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from statsmodels.stats.multicomp import pairwise_tukeyhsd
from statsmodels.formula.api import ols
from statsmodels.stats.anova import anova_lm
from scipy import stats


#데이터 불러오기
df = pd.read_csv("2021_gender.csv")
df_gender = df[['gender', 'age', 'years', 'education', 'income', 'job_status', 'marital_status', 'id_stance',
               'children', 'perceived_ssclass', 'newstype', 'news_form', 'location', 'position', 'gender_equity', 'gender_perception']]

#female dataframe 생성
female = (df_gender.gender == 1)
df_female = df_gender.loc[female, ['gender', 'age', 'years', 'education', 'income', 'job_status', 'marital_status', 'id_stance',
               'children', 'perceived_ssclass', 'newstype', 'news_form', 'location', 'position', 'gender_equity', 'gender_perception']]
df_female

#male dataframe 생성
male = (df_gender.gender == 0)
df_male = df_gender.loc[male, ['gender', 'age', 'years', 'education', 'income', 'job_status', 'marital_status', 'id_stance',
               'children', 'perceived_ssclass', 'newstype', 'news_form', 'location', 'position', 'gender_equity', 'gender_perception']]
df_male

#상관관계 나이(연차)와 성 불평등 문제 인식
age_gender_cor = stats.pearsonr(df_gender['age'], df_gender['gender_perception'])
print("%.2f, %.2f"% age_gender_cor) #-0.214, 0.000 나이가 많을수록 성 불평등 문제 심각하다고 느끼지 않는다.

years_gender_cor = stats.pearsonr(df_gender['years'], df_gender['gender_perception'])
print("%.2f, %.2f"%years_gender_cor) #-0.137, 0.000 연차가 많을수록 성 불평등 문제 심각하다고 느끼지 않는다.

#나이 lineplot
sns.set_style('darkgrid')
plt.rcParams["font.family"] = 'Yoon YGO 550_TT'
fig, ax = plt.subplots(figsize=(8,6))
sns.lineplot(x='age', y='gender_perception', ci=None, data=df_gender, ax=ax)
ax.set_title('나이와 성 불평등 문제 인식', x=0.5, y=1.05, fontsize=14)
#ax.text(x=1.0, y=1.01, s='r=-0.21, p=0.00', fontsize=10, alpha=0.75, ha='right', va='bottom', transform=ax.transAxes)
plt.xlabel('나이')
plt.ylabel('성 불평등 문제 인식')
plt.show()

#연차 lineplot
sns.set_style('darkgrid')
plt.rcParams["font.family"] = 'Yoon YGO 550_TT'
fig, ax = plt.subplots(figsize=(8,6))
sns.lineplot(x='years', y='gender_perception', ci=None, data=df_gender, ax=ax)
ax.set_title('연차와 성 불평등 문제 인식', x=0.5, y=1.05, fontsize=14)
#ax.text(x=1.0, y=1.01, s='r=-0.14, p=0.00', fontsize=10, alpha=0.75, ha='right', va='bottom', transform=ax.transAxes)
plt.xlabel('연차')
plt.ylabel('성 불평등 문제 인식')
plt.show()

#나이 세대별 범주화 (참조: https://dailyfreepress.com/2021/03/15/generation-names-explained/)
df_gender['birth'] = 2022 - df_gender['age']
df_gender.loc[df_gender['birth'] > 1997, 'birth'] = 4
df_gender.loc[df_gender['birth'] > 1981, 'birth'] = 3
df_gender.loc[df_gender['birth'] > 1965, 'birth'] = 2
df_gender.loc[df_gender['birth'] > 1946, 'birth'] = 1
df_gender['generation'] = df_gender['birth'].replace({1:'Baby Boomer', 2:'Generation X', 3:'Generation Y', 4:'Generation Z'})

#세대별 성 불평등 문제 인식 ANOVA, 사후분석
model = ols('gender_perception ~ C(generation)', df_gender).fit()
print(anova_lm(model)) #F=24.07 P < 0.05
hsd = pairwise_tukeyhsd(df_gender['gender_perception'], df_gender['generation'], alpha=0.05)
print(hsd.summary()) #BB - Y, BB - Z, X -Y

df_gender['generation'] = df_gender['birth'].replace({1:'Baby Boomer\n(1946-1964)', 2:'Generation X\n(1965-1980)',
                                                      3:'Generation Y\n(1981-1996)', 4:'Generation Z\n(1997-2012)'})

#세대별 boxplot
sns.set_style('darkgrid')
plt.rcParams["font.family"] = 'Yoon YGO 550_TT'
fig, ax = plt.subplots(figsize=(8,6))
sns.boxplot(x='generation', y='gender_perception', order=['Baby Boomer\n(1946-1964)', 'Generation X\n(1965-1980)',
                                                          'Generation Y\n(1981-1996)', 'Generation Z\n(1997-2012)'], data=df_gender, ax=ax)
ax.set_title('세대별 성 불평등 문제 인식', x=0.5, y=1.05, fontsize=14)
#ax.text(x=1.0, y=1.01, s='F=-24.07, p<0.05', fontsize=10, alpha=0.75, ha='right', va='bottom', transform=ax.transAxes)
plt.xlabel('세대 구분')
plt.ylabel('성 불평등 문제 인식')
plt.show()

#t-test 성별에 따른 성불평등 문제점 인식

#집단 구분 (남성/여성)
perception_male = df_gender.loc[df_gender['gender']=='남성', 'gender_perception']
perception_female = df_gender.loc[df_gender['gender']=='여성', 'gender_perception']

levene =stats.levene(perception_male, perception_female)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.633

ttest = stats.ttest_ind(perception_male, perception_female, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = -18.842 p = 0.000

#성별에 따른 성 불평등 문제 인식 barplot
sns.set_style('darkgrid')
colors = ["#95B7F0", "#F07674"]
sns.set_palette(sns.color_palette(colors))
plt.rcParams["font.family"] = 'Yoon YGO 550_TT'
fig, ax = plt.subplots(figsize=(8,6))
sns.barplot(x='gender', y='gender_perception', order=['남성','여성'], data=df_gender, ax=ax)
ax.set_title('성별간 성 불평등 문제 인식', x=0.5, y=1.05, fontsize=14)
#ax.text(x=1.0, y=1.01, s='t=-18.84, p<0.05', fontsize=10, alpha=0.75, ha='right', va='bottom', transform=ax.transAxes)
ax.annotate(text='평균=2.41', xy=(0, 1), xycoords='data', ha='center', fontsize=10, alpha=1)
ax.annotate(text='평균=3.11', xy=(1, 1), xycoords='data', ha='center', fontsize=10, alpha=1)
plt.xlabel('성별')
plt.ylabel('성 불평등 문제 인식')
plt.show()

#t-test 결혼 여부에 따른 성 불평등 문제 인식

#집단 구분 (미혼/기혼)
perception_married = df_gender.loc[df_gender['marital_status']=='기혼', 'gender_perception']
perception_single = df_gender.loc[df_gender['marital_status']=='미혼', 'gender_perception']

levene = stats.levene(perception_married, perception_single)
print("leveneresult(statistic = %.2f, pvalue = %.2f" %levene) #분산 동일성 검정 p = 0.001

ttest = stats.ttest_ind(perception_married, perception_single, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = -8.201, p = 0.000

#결혼 여부에 따른 성 불평등 문제 인식 barplot
sns.set_style('darkgrid')
plt.rcParams["font.family"] = 'Yoon YGO 550_TT'
fig, ax = plt.subplots(figsize=(8,6))
sns.barplot(x='marital_status', y='gender_perception', order=['미혼','기혼'], data=df_gender, ax=ax)
ax.set_title('결혼 여부에 따른 성 불평등 문제 인식', x=0.5, y=1.05, fontsize=14)
#ax.text(x=1.0, y=1.01, s='t=-8.20, p<0.05', fontsize=10, alpha=0.75, ha='right', va='bottom', transform=ax.transAxes)
ax.annotate(text='평균=2.81', xy=(0, 1), xycoords='data', ha='center', fontsize=10, alpha=1)
ax.annotate(text='평균=2.50', xy=(1, 1), xycoords='data', ha='center', fontsize=10, alpha=1)
plt.xlabel('결혼 여부')
plt.ylabel('성 불평등 문제 인식')
plt.show()
df_gender.rename(columns = {"gender":"성별"},inplace=True)

#직위에 따른 성별 비율 barplot
colors = ["#95B7F0", "#F07674"]
sns.set_palette(sns.color_palette(colors))
sns.set_style('darkgrid')
plt.rcParams["font.family"] = 'Yoon YGO 550_TT'
fig, ax = plt.subplots(figsize=(8,6))
plt.title('직위에 따른 성별 비율', x=0.5, y=1.05, fontsize=14)
sns.countplot(x='position', hue='성별', hue_order=['남성','여성'], data=df_gender, ax=ax)
ax.annotate(text='56%', xy=(-0.25, 620), xycoords='data', ha='left', fontsize=8, alpha=1)
ax.annotate(text='44%', xy=(0.15, 490), xycoords='data', ha='left', fontsize=8, alpha=1)
ax.annotate(text='74%', xy=(0.75, 316), xycoords='data', ha='left', fontsize=8, alpha=1)
ax.annotate(text='26%', xy=(1.15, 118), xycoords='data', ha='left', fontsize=8, alpha=1)
ax.annotate(text='86%', xy=(1.75, 220), xycoords='data', ha='left', fontsize=8, alpha=1)
ax.annotate(text='14%', xy=(2.15, 43), xycoords='data', ha='left', fontsize=8, alpha=1)
ax.annotate(text='89%', xy=(2.75, 115), xycoords='data', ha='left', fontsize=8, alpha=1)
ax.annotate(text='11%', xy=(3.15, 23), xycoords='data', ha='left', fontsize=8, alpha=1)
ax.annotate(text='93%', xy=(3.75, 149), xycoords='data', ha='left', fontsize=8, alpha=1)
ax.annotate(text='7%', xy=(4.15, 20), xycoords='data', ha='left', fontsize=8, alpha=1)
plt.xlabel('직위')
plt.show()

#직위와 성별 crosstab & chi-square
df_gender['position_r'] = df_gender['position'].replace({'국장':5, '부국장':4, '부장':3, '차장':2, '평기자':1})
pd.crosstab(df_gender["gender"], df_gender["position_r"])
contingency = pd.crosstab(df_gender["gender"], df_gender["position_r"])
print(contingency)
chi, p , dof, expected = stats.chi2_contingency(contingency)
print(f"카이스퀘어 검정통계량: {chi:.2f}",
      f"p-value(0.05): {p:.2f}",
      f"자유도: {dof}", sep = "\n" ) #184.854, 0.000, 4

#직위에 따른 성 불평등 문제 인식 ANOVA, 사후분석
model = ols('gender_perception ~ C(position)', df_gender).fit()
print(anova_lm(model)) #F=17.13 p<0.05

hsd = pairwise_tukeyhsd(df_gender['gender_perception'], df_gender['position'], alpha=0.05)
print(hsd.summary()) #평기자 - 국장 / 평기자 - 부국장 / 평기자 - 부장 / 평기자 - 차장

#직위에 따른 성 불평등 문제 인식 boxplot
colors = ["#95B7F0", "#BD86D9", "#F07B71"]
sns.set_palette(sns.color_palette(colors))
sns.set_style('darkgrid')
plt.rcParams["font.family"] = 'Yoon YGO 550_TT'
fig, ax = plt.subplots(figsize=(8,6))
sns.boxplot(x='position', y='gender_perception', data=df_gender)
ax.set_title('직위별 성 불평등 문제점 인식', x=0.5, y=1.05, fontsize=14)
#ax.text(x=1.0, y=1.01, s='F=17.13, p<0.05', fontsize=10, alpha=0.75, ha='right', va='bottom', transform=ax.transAxes)
plt.xlabel('직위')
plt.ylabel('성 불평등 문제 인식')
plt.show()
#keep

#직위 집단 구분 (국장,부국장/부장,차장/평기자)
df_gender['position_group'] = df_gender['position'].replace({'국장':'임원','부국장':'임원','부장':'관리직','차장':'관리직','평기자':'사원'})
print(df_gender['position_group'].value_counts())

#직위 그룹에 따른 성 불평등 문제 인식 ANOVA, 사후분석
model = ols('gender_perception ~ C(position_group)', df_gender).fit()
print(anova_lm(model)) #F=32.54, p > 0.05

hsd = pairwise_tukeyhsd(df_gender['gender_perception'], df_gender['position_group'], alpha=0.05)
print(hsd.summary()) #관리직 - 사원 / 임원 - 사원 차이 있음

#직위 그룹에 따른 성 불평등 문제 인식 barplot
sns.set_style('darkgrid')
colors = ["#95B7F0", "#BD86D9", "#F07B71"]
sns.set_palette(sns.color_palette(colors))
plt.rcParams["font.family"] = 'Yoon YGO 550_TT'
fig, ax = plt.subplots(figsize=(8,6))
sns.barplot(x='position_group', y='gender_perception', data=df_gender)
ax.set_title('직위 그룹별 성 불평등 문제점 인식', x=0.5, y=1.05, fontsize=14)
ax.text(x=1.0, y=1.00, s='평기자: 사원 / 관리직: 부장, 차장/ 임원: 국장, 부국장', fontsize=8, alpha=0.75, ha='right', va='bottom', transform=ax.transAxes)
plt.xlabel('직위 그룹 구분')
plt.ylabel('성 불평등 문제 인식')
plt.show()

####################################################################################################################

#여성인 경우, 나이와 성 불평등 문제 인식 간 상관관계
age_female_cor = stats.pearsonr(df_female['age'], df_female['gender_perception'])
print("%.2f, %.2f"% age_female_cor)

#나이와 성 불평등 문제 인식 linplot (남성/여성)
sns.set_style('darkgrid')
plt.rcParams["font.family"] = 'Yoon YGO 550_TT'
colors = ["#95B7F0", "#F07674"]
sns.set_palette(sns.color_palette(colors))
fig, ax = plt.subplots(figsize=(8,6))
sns.lineplot(x='age', y='gender_perception', label='남성인 경우', ci=None, data=df_male)
sns.lineplot(x='age', y='gender_perception', label='여성인 경우', ci=None, data=df_female)
ax.set_title('나이에 따른 성 불평등 문제 인식 (남성인 경우 / 여성인 경우)', x=0.5, y=1.05, fontsize=14)
#ax.text(x=1.0, y=1.01, s='r=-0.13, p<0.05', fontsize=10, alpha=0.75, ha='right', va='bottom', transform=ax.transAxes)
plt.xlabel('나이')
plt.ylabel('성 불평등 문제 인식')
plt.legend
plt.show()

#남자 여자 구분

#df_female 나이 세대별 범주화
df_female['birth'] = 2022 - df_female['age']
df_female.loc[df_female['birth'] > 1997, 'birth'] = 4
df_female.loc[df_female['birth'] > 1981, 'birth'] = 3
df_female.loc[df_female['birth'] > 1965, 'birth'] = 2
df_female.loc[df_female['birth'] > 1946, 'birth'] = 1
df_female['generation'] = df_female['birth'].replace({1:'Baby Boomer', 2:'Generation X', 3:'Generation Y', 4:'Generation Z'})

#여성인 경우, 세대에 따른 성 불평등 문제 인식 ANOVA, 사후분석
model = ols('gender_perception ~ C(generation)', df_female).fit()
print(anova_lm(model)) #p값 0.046 으로 나오는데 사후분석 실시했을 때 모두 False
hsd = pairwise_tukeyhsd(df_female['gender_perception'], df_female['generation'], alpha=0.05)
print(hsd.summary())

sns.barplot(x='generation', y='gender_perception', data=df_female, order=['Baby Boomer', 'Generation X', 'Generation Y', 'Generation Z'])
plt.rc('font', family='AppleGothic')
plt.show()

#집단 구분 (미혼, 기혼)
perception_married = df_female.loc[df_female['marital_status']=='기혼', 'gender_perception']
perception_single = df_female.loc[df_female['marital_status']=='미혼', 'gender_perception']

levene = stats.levene(perception_married, perception_single)
print("leveneresult(statistic = %.2f, pvalue = %.2f" %levene) #분산 동일성 검정 p = 0.08

ttest = stats.ttest_ind(perception_married, perception_single, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = -2.12, p = 0.03

plt.rcParams["font.family"] = 'Yoon YGO 550_TT'
fig, ax = plt.subplots(figsize=(15,6))
ax1 = fig.add_subplot(1, 2, 1)
ax2 = fig.add_subplot(1, 2, 2)
sns.barplot(x='marital_status', y='gender_perception', order=['미혼','기혼'], data=df_male, ax=ax1)
sns.barplot(x='marital_status', y='gender_perception', order=['미혼','기혼'], data=df_female, ax=ax2)
ax.set_title('결혼 여부에 따른 성 불평등 문제 인식 그래프 (남성인 경우 / 여성인 경우)', x=0.5, y=1.05, fontsize=14)
#ax.text(x=0, y=1.01, s='t=-8.20, p<0.05', fontsize=10, alpha=0.75, ha='left', va='bottom', transform=ax.transAxes)
#ax.text(x=1.0, y=1.01, s='t=-2.12, p<0.05', fontsize=10, alpha=0.75, ha='right', va='bottom', transform=ax.transAxes)
ax1.set_xlabel('결혼 여부')
ax1.set_ylabel('성 불평등 문제 인식')
ax2.set_xlabel('결혼 여부')
ax2.set_ylabel('성 불평등 문제 인식')
plt.show() #x축 숫자, y축 숫자, 중간 구분 등 수정 필요

#ANOVA (성평등 문제점과 직위)
model = ols('gender_perception ~ C(position)', df_female).fit()
print(anova_lm(model))

#사후분석
hsd = pairwise_tukeyhsd(df_female['gender_perception'], df_female['position'], alpha=0.05)
print(hsd.summary())

#position 집단 구분 (국장,부국장,부장,차장/평기자)
df_gender['position_group'] = df_gender['position'].replace({'국장':'간부','부국장':'간부','부장':'간부','차장':'간부','평기자':'사원'})

df_female['position_group'] = df_female['position'].replace({'국장':'간부','부국장':'간부','부장':'간부','차장':'간부','평기자':'사원'})
df_female['position_group'].value_counts()

df_male['position_group'] = df_male['position'].replace({'국장':'간부','부국장':'간부','부장':'간부','차장':'간부','평기자':'사원'})
df_male['position_group'].value_counts()

female_group_a = df_female.loc[df_female['position_group']=='간부', 'gender_perception']
female_group_b = df_female.loc[df_female['position_group']=='사원', 'gender_perception']

male_group_a = df_male.loc[df_male['position_group']=='간부', 'gender_perception']
male_group_b = df_male.loc[df_male['position_group']=='사원', 'gender_perception']

sns.boxplot(x='position_group', y='gender_perception', data=df_male)
plt.show()

#간부와 사원 간 성 불평등 t-test
levene =stats.levene(female_group_a, female_group_b)
print("levenresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정
ttest = stats.ttest_ind(female_group_a, female_group_b, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t-value = -2.21, p-value = 0.03

levene =stats.levene(male_group_a, male_group_b)
print("levenresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정
ttest = stats.ttest_ind(male_group_a, male_group_b, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t-value = -2.39, p-value = 0.02

df_gender.rename(columns = {"gender":"성별"},inplace=True)

sns.set_style('darkgrid')
colors = ["#95B7F0", "#F07B71"]
sns.set_palette(sns.color_palette(colors))
plt.rcParams["font.family"] = 'Yoon YGO 550_TT'
fig, ax = plt.subplots(figsize=(8,6))
sns.boxplot(x='position_group', y='gender_perception', hue="성별", hue_order=['남성','여성'], data=df_gender)
ax.set_title('직위 그룹별 성 불평등 문제점 인식 (남/녀 구분)', x=0.5, y=1.05, fontsize=14)
#ax.text(x=1.0, y=1.01, s='t= -2.21, p<0.05', fontsize=10, alpha=0.75, ha='right', va='bottom', transform=ax.transAxes)
plt.xlabel('직위 그룹 구분')
plt.ylabel('성 불평등 문제 인식')
plt.show()
 
```