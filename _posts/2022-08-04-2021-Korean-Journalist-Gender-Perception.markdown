---
layout: post
title: 2021 한국의 언론인 - 언론 내 성 불평등 인식
image: 13.jpg
date: 2022-08-03 18:23:20 +0200
tags: [Project, 2021 Korean Journalist]
categories: Project
---
한국언론진흥재단은 매년 언론인을 대상으로 통계조사를 진행하고 있다. 한국언론진흥재단의 [언론인 조사]는 국내 유일의 언론인 대상 통계 조사다. 

본 연구자는 [2021 한국의 언론인] 데이터에서 언론사의 성 불평등 인식을 집중하여 살펴보았다. 최근 몇 년간 젠더 갈등이 큰 이슈로 대두되었다. 많은 사람들이 관심을 가지고 있는 주제인 만큼 언론 역시 집중해서 보도하는 경향을 보였다. 이처럼 갈등이 심화되고 있는 상황에서 언론은 중립적인 입장을 취하는 것이 중요하다. 따라서 현재 언론 내에서의 성 불평등 인식을 중점으로 살펴보았다.

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

#t-test 결혼 여부에 따른 성평등 문제점

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