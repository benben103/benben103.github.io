---
layout: post
title: 2021 언론 수용자 인식 조사 - Z세대와 베이비 붐 세대의 디지털 미디어 이용 행태
image: 9.jpg
date: 2022-08-03 17:50:18 +0200
tags: [Project, 2021 Media Audience]
categories: Project
---
본 연구는 한국언론진흥재단의 2021 언론 수용자 인식 조사 보고서를 활용하여 Z세대와 베이비 붐 세대의 디지털 미디어 이용 행태를 탐색하였다. 

디지털 미디어 기술이 보편화 되면서 사람들의 디지털 미디어/콘텐츠 이용 또한 늘어났다. 전통 미디어로 분류된 텔레비전, 신문, 라디오 등의 매체 소비가 줄어들고, 포털, 메신저 서비스, SNS, 온라인 동영상 플랫폼 등의 디지털 미디어 이용이 증가하였다(한국언론진흥재단, 2019). 
디지털 미디어 보편화는 다양한 정보를 제공하지만, 세대 별 미디어 이용 능력이 다르기 때문에 세대 간 미디어 이용 격차가 나타날 수 있다(황용석∙박남수∙이현주∙이원태, 2012). 즉, 미디어와 콘텐츠 이용방식과 수준은 연령과 세대에 따라 다르게 나타나고 이로 인해 세대 간 갈등이 유발 또는 심화 될 수 있다.

---

[2021 언론수용자 조사 - 한국언론진흥재단](https://www.kpf.or.kr/front/research/consumerDetail.do?miv_pageNo=&miv_pageSize=&total_cnt=&LISTOP=&mode=W&seq=592381&link_g_topmenu_id=&link_g_submenu_id=&link_g_homepage=F&reg_stadt=&reg_enddt=&searchkey=all1&searchtxt=)
{% highlight markdown %}
* 조사 대상: 전국 만 19세 이상 국민 5,010명
* 조사 방법: 가구 방문을 통한 컴퓨터 이용 대면면접조사 (CAPI)
* 조사 기간: 2021년 5월 31일 ~ 7월 11일
자세한 내용은 홈페이지 참조
{% endhighlight %}

---

#### 데이터 불러오기 및 변수 설정

```python 
import pandas as pd

df_2021 = pd.read_csv("2021 언론수용자 조사 DATA_공개용_최종.csv")

df_2021.rename(columns={'SQ1':'거주지역','SQ2':'도시규모','SQ3':'성별','SQ4':'연령'}, inplace=True) # 변수 이름 변경
df_2021['성별'].replace({1:0, 2:1}, inplace=True) # 변수 값 변경

df_2021['세대'] = np.nan
df_2021.loc[df_2021['연령'] <= 26,'세대']='young'
df_2021.loc[(df_2021['연령'] >= 58) & (df_2021['연령'] <= 66), '세대']='old'
```

pandas 모듈을 이용하여 csv 파일을 불러올 수 있다. csv 파일의 변수 이름과 값을 사용하기 쉽게 변경하는 과정을 거치는 것이 좋다. 전체 코드 중 일부를 가져왔다.

데이터프레임.rename(columns={'기존 이름':'새로운 이름'}, inplace=True)은 열 이름을 바꿔주는 명령어다.

데이터프레임['변수 이름'].replace({'기존 값':'바꿔줄 값'}, inplace=True)는 해당 변수의 값을 변경하는 명령어다.

세대 구분을 위해 연령 변수를 이용하여 세대 변수를 생성하였다. Z세대는 2021년을 기준으로 26세 이하로 설정하였고, Baby Boomer세대는 58세에서 66세로 설정하였다. 변수의 값을 변경하거나 새로 생성할 때, 조건을 추가하려면 위와 같은 방법으로 실행하면 된다.



---

#### 전체 Code
```python 
import numpy as np
import pandas as pd
import pingouin as pg #Cronbach's alpha
from factor_analyzer.factor_analyzer import calculate_bartlett_sphericity
from statsmodels.stats.multicomp import pairwise_tukeyhsd
from statsmodels.formula.api import ols
from statsmodels.stats.anova import anova_lm
import matplotlib.pyplot as plt
from scipy import stats

df_2021 = pd.read_csv("2021 언론수용자 조사 DATA_공개용_최종.csv")

#인구통계학
df_2021.rename(columns={'SQ1':'거주지역','SQ2':'도시규모','SQ3':'성별','SQ4':'연령'}, inplace=True)
df_2021['거주지역'].replace({1:1, 2:0, 3:0, 4:0, 5:0, 6:0, 7:0, 8:0, 9:0, 10:0, 11:0, 12:0, 13:0, 14:0, 15:0, 16:0, 17:0}, inplace=True) #서울 1 나머지 0
#df_2021['거주지역'].value_counts()
df_2021['도시규모'].replace({1:3, 3:1}, inplace=True) # 1.대도시, 2. 중소도시, 3.군 -> 1.군, 2.중소도시, 3.대도시
df_2021['성별'].replace({1:0, 2:1}, inplace=True) # 0 = 남성 (2488) , 1 = 여성 (2522)

df_2021['세대'] = np.nan
df_2021.loc[df_2021['연령'] <= 26,'세대']='young'
df_2021.loc[(df_2021['연령'] >= 58) & (df_2021['연령'] <= 66), '세대']='old'


#뉴스이용미디어
df_2021.rename(columns={'Q1':'뉴스이용미디어'}, inplace=True)
df_2021['뉴스이용미디어'].replace({1:'종이신문', 2:'텔레비전', 3:'라디오', 4:'잡지', 5:'인터넷포털',
                            6:'뉴스사이트', 7:'SNS', 8:'메신저', 9:'동영상플랫폼', 10:'인공지능스피커', 11:'뉴스레터', 12:'없음'}, inplace=True)
#df_2021['뉴스이용미디어'].value_counts()

#인터넷 포털 및 언론사 사이트 이용
df_2021['포털이용일수_스마트폰'] = df_2021[['Q38x1_1', 'Q38x2_1']].sum(axis=1) #mean=4.94
df_2021.loc[(df_2021['세대'] == 'old') | (df_2021['세대'] == 'young'), '포털이용일수_스마트폰'].mean()

df_2021.rename(columns={'Q43_1':'포털_국제', 'Q43_2':'포털_정치', 'Q43_3':'포털_사회', 'Q43_4':'포털_경제', 'Q43_5':'포털_교육', 'Q43_6':'포털_과학',
                        'Q43_7':'포털_남북', 'Q43_8':'포털_생활', 'Q43_9':'포털_연예', 'Q43_10':'포털_스포츠', 'Q43_11':'포털_지역정보', 'Q43_12':'포털_사설'}, inplace=True)
df_2021['포털_국제'].replace({' ':0, '1':1}, inplace=True) #counts 122 / 179
df_2021['포털_정치'].replace({' ':0, '2':1}, inplace=True) #counts 186 / 387
df_2021['포털_사회'].replace({' ':0, '3':1}, inplace=True) #counts 350 / 489
df_2021['포털_경제'].replace({' ':0, '4':1}, inplace=True) #counts 233 / 400
df_2021['포털_교육'].replace({' ':0, '5':1}, inplace=True) #counts 120 / 97
df_2021['포털_과학'].replace({' ':0, '6':1}, inplace=True) #counts 84 / 84
df_2021['포털_남북'].replace({' ':0, '7':1}, inplace=True) #counts 43 / 93
df_2021['포털_생활'].replace({' ':0, '8':1}, inplace=True) #counts 229 / 324
df_2021['포털_연예'].replace({' ':0, '9':1}, inplace=True) #counts 306 / 205
df_2021['포털_스포츠'].replace({' ':0, '10':1}, inplace=True) #counts 150 / 162
df_2021['포털_지역정보'].replace({' ':0, '11':1}, inplace=True) #counts 34 / 105
df_2021['포털_사설'].replace({' ':0, '12':1}, inplace=True) #counts 25 / 52

df_2021['포털주제수'] = df_2021[['포털_국제', '포털_정치', '포털_사회', '포털_경제', '포털_교육', '포털_과학',
                        '포털_남북', '포털_생활', '포털_연예', '포털_스포츠', '포털_지역정보', '포털_사설']].sum(axis=1)

df_2021.rename(columns={'Q44_1':'포털뉴스활동_1', 'Q44_2':'포털뉴스활동_2', 'Q44_3':'포털뉴스활동_3'}, inplace=True) # 1-공유, 2-공감표시, 3-댓글
df_2021['포털뉴스활동_1'].replace({2:0}, inplace=True) #없다 = 0, 있다 = 1/ counts 419
df_2021['포털뉴스활동_2'].replace({2:0}, inplace=True) #counts 508
df_2021['포털뉴스활동_3'].replace({2:0}, inplace=True) #counts 275

evaluation = df_2021[['포털뉴스활동_1', '포털뉴스활동_2', '포털뉴스활동_3']]
pg.cronbach_alpha(data=evaluation)
df_2021['포털뉴스활동'] = df_2021[['포털뉴스활동_1', '포털뉴스활동_2', '포털뉴스활동_3']].sum(axis=1) #mean=0.24 a=0.68
df_2021.loc[(df_2021['세대'] == 'old') | (df_2021['세대'] == 'young'), '포털뉴스활동'].mean()

#t-test 포털 이용량
ttest_left = df_2021.loc[df_2021['세대']=='young', '포털이용일수_스마트폰'] #4.61
ttest_right = df_2021.loc[df_2021['세대']=='old', '포털이용일수_스마트폰'] #2.16
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 35.73 p = 0.00 ***

#t-test 포털 뉴스 주제
ttest_left = df_2021.loc[df_2021['세대']=='young', '포털_국제'] #0.27
ttest_right = df_2021.loc[df_2021['세대']=='old', '포털_국제'] #0.16
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 5.28 p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='young', '포털_정치'] #0.43
ttest_right = df_2021.loc[df_2021['세대']=='old', '포털_정치'] #0.33
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 3.96 p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='young', '포털_사회'] #0.76
ttest_right = df_2021.loc[df_2021['세대']=='old', '포털_사회'] #0.41
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 16.27 p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='young', '포털_경제'] #0.53
ttest_right = df_2021.loc[df_2021['세대']=='old', '포털_경제'] #0.34
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 7.97 p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='young', '포털_교육'] #0.25
ttest_right = df_2021.loc[df_2021['세대']=='old', '포털_교육'] #0.08
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 8.73 p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='young', '포털_과학'] #0.19
ttest_right = df_2021.loc[df_2021['세대']=='old', '포털_과학'] #0.07
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 8.73 p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='young', '포털_남북'] #0.10
ttest_right = df_2021.loc[df_2021['세대']=='old', '포털_남북'] #0.08
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.25
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 1.14 p = 0.25

ttest_left = df_2021.loc[df_2021['세대']=='young', '포털_생활'] #0.52
ttest_right = df_2021.loc[df_2021['세대']=='old', '포털_생활'] #0.27
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 10.66 p = 0.00

ttest_left = df_2021.loc[df_2021['세대']=='young', '포털_연예'] #0.66
ttest_right = df_2021.loc[df_2021['세대']=='old', '포털_연예'] #0.16
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 22.63 p = 0.00

ttest_left = df_2021.loc[df_2021['세대']=='young', '포털_스포츠'] #0.33
ttest_right = df_2021.loc[df_2021['세대']=='old', '포털_스포츠'] #0.13
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 9.57 p = 0.00

ttest_left = df_2021.loc[df_2021['세대']=='young', '포털_지역정보'] #0.08
ttest_right = df_2021.loc[df_2021['세대']=='old', '포털_지역정보'] #0.08
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.93
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 0.09 p = 0.93

ttest_left = df_2021.loc[df_2021['세대']=='young', '포털_사설'] #0.06
ttest_right = df_2021.loc[df_2021['세대']=='old', '포털_사설'] #0.05
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.23
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 1.19 p = 0.23

ttest_left = df_2021.loc[df_2021['세대']=='young', '포털주제수'] #4.17
ttest_right = df_2021.loc[df_2021['세대']=='old', '포털주제수'] #2.16
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.01
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 16.29 p = 0.00

#t-test 포털 미디어 참여
ttest_left = df_2021.loc[df_2021['세대']=='young', '포털뉴스활동'] #0.47
ttest_right = df_2021.loc[df_2021['세대']=='old', '포털뉴스활동'] #0.05
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 11.43 p = 0.00 ***


#메신저 서비스 이용
df_2021.rename(columns={'Q47':'메신저이용여부', 'Q50':'메신저뉴스이용여부'}, inplace=True)
df_2021['메신저이용여부'].replace({2:0}, inplace=True) # 이용 안했다 = 0, 이용했다 = 1 (4088)
df_2021['메신저뉴스이용여부'].replace({2:0}, inplace=True) # 이용 안했다 = 0, 이용했다 = 1 (777)

df_2021['메신저이용일수'] = df_2021[['Q48x1', 'Q48x2']].sum(axis=1) #mean=5.31
df_2021.loc[(df_2021['세대'] == 'old') | (df_2021['세대'] == 'young'), '메신저이용일수'].mean()

df_2021.rename(columns={'Q53_1':'메신저_국제', 'Q53_2':'메신저_정치', 'Q53_3':'메신저_사회', 'Q53_4':'메신저_경제', 'Q53_5':'메신저_교육', 'Q53_6':'메신저_과학',
                        'Q53_7':'메신저_남북', 'Q53_8':'메신저_생활', 'Q53_9':'메신저_연예', 'Q53_10':'메신저_스포츠', 'Q53_11':'메신저_지역정보', 'Q53_12':'메신저_사설'}, inplace=True)
df_2021['메신저_국제'].replace({' ':0, '1':1}, inplace=True) #counts 120
df_2021['메신저_정치'].replace({' ':0, '2':1}, inplace=True) #counts 252
df_2021['메신저_사회'].replace({' ':0, '3':1}, inplace=True) #counts 540
df_2021['메신저_경제'].replace({' ':0, '4':1}, inplace=True) #counts 323
df_2021['메신저_교육'].replace({' ':0, '5':1}, inplace=True) #counts 126
df_2021['메신저_과학'].replace({' ':0, '6':1}, inplace=True) #counts 96
df_2021['메신저_남북'].replace({' ':0, '7':1}, inplace=True) #counts 66
df_2021['메신저_생활'].replace({' ':0, '8':1}, inplace=True) #counts 298
df_2021['메신저_연예'].replace({' ':0, '9':1}, inplace=True) #counts 378
df_2021['메신저_스포츠'].replace({' ':0, '10':1}, inplace=True) #counts 153
df_2021['메신저_지역정보'].replace({' ':0, '11':1}, inplace=True) #counts 50
df_2021['메신저_사설'].replace({' ':0, '12':1}, inplace=True) #counts 32

df_2021['메신저주제수'] = df_2021[['메신저_국제', '메신저_정치', '메신저_사회', '메신저_경제', '메신저_교육', '메신저_과학',
                        '메신저_남북', '메신저_생활', '메신저_연예', '메신저_스포츠', '메신저_지역정보', '메신저_사설']].sum(axis=1)

df_2021.rename(columns={'Q54_1':'메신저뉴스활동_1', 'Q54_2':'메신저뉴스활동_2', 'Q54_3':'메신저뉴스활동_3'}, inplace=True) # 1-공유, 2-공감표시, 3-댓글
df_2021['메신저뉴스활동_1'].replace({2:0}, inplace=True) #없다 = 0, 있다 = 1/ counts 220
df_2021['메신저뉴스활동_2'].replace({2:0}, inplace=True) #counts 175
df_2021['메신저뉴스활동_3'].replace({2:0}, inplace=True) #counts 105
df_2021['메신저뉴스활동'] = df_2021[['메신저뉴스활동_1', '메신저뉴스활동_2', '메신저뉴스활동_3']].sum(axis=1) #mean=0.10
df_2021.loc[(df_2021['세대'] == 'old') | (df_2021['세대'] == 'young'), '메신저뉴스활동'].mean()

#t-test 메신저 이용량
ttest_left = df_2021.loc[df_2021['세대']=='young', '메신저이용일수'] #4.72
ttest_right = df_2021.loc[df_2021['세대']=='old', '메신저이용일수'] #2.54
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

#t-test 메신저 뉴스 주제
ttest_left = df_2021.loc[df_2021['세대']=='young', '메신저_국제'] #0.20
ttest_right = df_2021.loc[df_2021['세대']=='old', '메신저_국제'] #0.02
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '메신저_정치'] #0.20
ttest_right = df_2021.loc[df_2021['세대']=='old', '메신저_정치'] #0.02
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '메신저_사회'] #0.20
ttest_right = df_2021.loc[df_2021['세대']=='old', '메신저_사회'] #0.02
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '메신저_경제'] #0.20
ttest_right = df_2021.loc[df_2021['세대']=='old', '메신저_경제'] #0.02
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '메신저_교육'] #0.20
ttest_right = df_2021.loc[df_2021['세대']=='old', '메신저_교육'] #0.02
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '메신저_과학'] #0.20
ttest_right = df_2021.loc[df_2021['세대']=='old', '메신저_과학'] #0.02
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '메신저_남북'] #0.20
ttest_right = df_2021.loc[df_2021['세대']=='old', '메신저_남북'] #0.02
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '메신저_생활'] #0.20
ttest_right = df_2021.loc[df_2021['세대']=='old', '메신저_생활'] #0.02
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '메신저_연예'] #0.20
ttest_right = df_2021.loc[df_2021['세대']=='old', '메신저_연예'] #0.02
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '메신저_스포츠'] #0.20
ttest_right = df_2021.loc[df_2021['세대']=='old', '메신저_스포츠'] #0.02
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '메신저_지역정보'] #0.20
ttest_right = df_2021.loc[df_2021['세대']=='old', '메신저_지역정보'] #0.02
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '메신저_사설'] #0.20
ttest_right = df_2021.loc[df_2021['세대']=='old', '메신저_사설'] #0.02
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '메신저주제수'] #0.20
ttest_right = df_2021.loc[df_2021['세대']=='old', '메신저주제수'] #0.02
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

#t-test 메신저 참여
ttest_left = df_2021.loc[df_2021['세대']=='young', '메신저뉴스활동'] #0.20
ttest_right = df_2021.loc[df_2021['세대']=='old', '메신저뉴스활동'] #0.02
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)


#SNS 이용
df_2021.rename(columns={'Q55':'SNS이용여부', 'Q58':'SNS뉴스이용여부'}, inplace=True)
df_2021['SNS이용여부'].replace({2:0}, inplace=True) # 이용 안했다 = 0, 이용했다 = 1 (2195)
df_2021['SNS뉴스이용여부'].replace({2:0}, inplace=True) # 이용 안했다 = 0, 이용했다 = 1 (547)

df_2021['SNS이용일수'] = df_2021[['Q56x1', 'Q56x2']].sum(axis=1) #mean=2.24
df_2021.loc[(df_2021['세대'] == 'old') | (df_2021['세대'] == 'young'), 'SNS이용일수'].mean()

df_2021.rename(columns={'Q61_1':'SNS_국제', 'Q61_2':'SNS_정치', 'Q61_3':'SNS_사회', 'Q61_4':'SNS_경제', 'Q61_5':'SNS_교육', 'Q61_6':'SNS_과학',
                        'Q61_7':'SNS_남북', 'Q61_8':'SNS_생활', 'Q61_9':'SNS_연예', 'Q61_10':'SNS_스포츠', 'Q61_11':'SNS_지역정보', 'Q61_12':'SNS_사설'}, inplace=True)
df_2021['SNS_국제'].replace({' ':0, '1':1}, inplace=True) #counts 98
df_2021['SNS_정치'].replace({' ':0, '2':1}, inplace=True) #counts 195
df_2021['SNS_사회'].replace({' ':0, '3':1}, inplace=True) #counts 382
df_2021['SNS_경제'].replace({' ':0, '4':1}, inplace=True) #counts 247
df_2021['SNS_교육'].replace({' ':0, '5':1}, inplace=True) #counts 81
df_2021['SNS_과학'].replace({' ':0, '6':1}, inplace=True) #counts 56
df_2021['SNS_남북'].replace({' ':0, '7':1}, inplace=True) #counts 52
df_2021['SNS_생활'].replace({' ':0, '8':1}, inplace=True) #counts 185
df_2021['SNS_연예'].replace({' ':0, '9':1}, inplace=True) #counts 272
df_2021['SNS_스포츠'].replace({' ':0, '10':1}, inplace=True) #counts 109
df_2021['SNS_지역정보'].replace({' ':0, '11':1}, inplace=True) #counts 28
df_2021['SNS_사설'].replace({' ':0, '12':1}, inplace=True) #counts 16

df_2021['SNS주제수'] = df_2021[['SNS_국제', 'SNS_정치', 'SNS_사회', 'SNS_경제', 'SNS_교육', 'SNS_과학',
                        'SNS_남북', 'SNS_생활', 'SNS_연예', 'SNS_스포츠', 'SNS_지역정보', 'SNS_사설']].sum(axis=1)

df_2021.rename(columns={'Q62_1':'SNS뉴스활동_1', 'Q62_2':'SNS뉴스활동_2', 'Q62_3':'SNS뉴스활동_3'}, inplace=True) # 1-공유, 2-공감표시, 3-댓글
df_2021['SNS뉴스활동_1'].replace({2:0}, inplace=True) #없다 = 0, 있다 = 1/ counts 171
df_2021['SNS뉴스활동_2'].replace({2:0}, inplace=True) #counts 188
df_2021['SNS뉴스활동_3'].replace({2:0}, inplace=True) #counts 104
df_2021['SNS뉴스활동'] = df_2021[['SNS뉴스활동_1', 'SNS뉴스활동_2', 'SNS뉴스활동_3']].sum(axis=1) #mean=0.09
df_2021.loc[(df_2021['세대'] == 'old') | (df_2021['세대'] == 'young'), 'SNS뉴스활동'].mean()

#t-test SNS 이용량
ttest_left = df_2021.loc[df_2021['세대']=='young', 'SNS이용일수']
ttest_right = df_2021.loc[df_2021['세대']=='old', 'SNS이용일수']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

#t-test SNS 뉴스 주제
ttest_left = df_2021.loc[df_2021['세대']=='young', 'SNS_국제']
ttest_right = df_2021.loc[df_2021['세대']=='old', 'SNS_국제']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', 'SNS_정치']
ttest_right = df_2021.loc[df_2021['세대']=='old', 'SNS_정치']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', 'SNS_사회']
ttest_right = df_2021.loc[df_2021['세대']=='old', 'SNS_사회']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', 'SNS_경제']
ttest_right = df_2021.loc[df_2021['세대']=='old', 'SNS_경제']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', 'SNS_교육']
ttest_right = df_2021.loc[df_2021['세대']=='old', 'SNS_교육']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', 'SNS_과학']
ttest_right = df_2021.loc[df_2021['세대']=='old', 'SNS_과학']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', 'SNS_남북']
ttest_right = df_2021.loc[df_2021['세대']=='old', 'SNS_남북']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', 'SNS_생활']
ttest_right = df_2021.loc[df_2021['세대']=='old', 'SNS_생활']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', 'SNS_연예']
ttest_right = df_2021.loc[df_2021['세대']=='old', 'SNS_연예']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', 'SNS_스포츠']
ttest_right = df_2021.loc[df_2021['세대']=='old', 'SNS_스포츠']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', 'SNS_지역정보']
ttest_right = df_2021.loc[df_2021['세대']=='old', 'SNS_지역정보']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', 'SNS_사설']
ttest_right = df_2021.loc[df_2021['세대']=='old', 'SNS_사설']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', 'SNS주제수']
ttest_right = df_2021.loc[df_2021['세대']=='old', 'SNS주제수']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

#t-test SNS 참여
ttest_left = df_2021.loc[df_2021['세대']=='young', 'SNS뉴스활동']
ttest_right = df_2021.loc[df_2021['세대']=='old', 'SNS뉴스활동']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)



#온라인 동영상 플랫폼
df_2021.rename(columns={'Q63':'동영상이용여부', 'Q66':'동영상뉴스이용여부'}, inplace=True)
df_2021['동영상이용여부'].replace({2:0}, inplace=True) # 이용 안했다 = 0, 이용했다 = 1 (3435)
df_2021['동영상뉴스이용여부'].replace({2:0}, inplace=True) # 이용 안했다 = 0, 이용했다 = 1 (1288)

df_2021['동영상이용일수'] = df_2021[['Q64x1', 'Q64x2']].sum(axis=1) #mean=3.77
df_2021.loc[(df_2021['세대'] == 'old') | (df_2021['세대'] == 'young'), '동영상이용일수'].mean()

df_2021.rename(columns={'Q69_1':'동영상_국제', 'Q69_2':'동영상_정치', 'Q69_3':'동영상_사회', 'Q69_4':'동영상_경제', 'Q69_5':'동영상_교육', 'Q69_6':'동영상_과학',
                        'Q69_7':'동영상_남북', 'Q69_8':'동영상_생활', 'Q69_9':'동영상_연예', 'Q69_10':'동영상_스포츠', 'Q69_11':'동영상_지역정보', 'Q69_12':'동영상_사설'}, inplace=True)
df_2021['동영상_국제'].replace({' ':0, '1':1}, inplace=True) #counts 284
df_2021['동영상_정치'].replace({' ':0, '2':1}, inplace=True) #counts 670
df_2021['동영상_사회'].replace({' ':0, '3':1}, inplace=True) #counts 1033
df_2021['동영상_경제'].replace({' ':0, '4':1}, inplace=True) #counts 644
df_2021['동영상_교육'].replace({' ':0, '5':1}, inplace=True) #counts 153
df_2021['동영상_과학'].replace({' ':0, '6':1}, inplace=True) #counts 136
df_2021['동영상_남북'].replace({' ':0, '7':1}, inplace=True) #counts 101
df_2021['동영상_생활'].replace({' ':0, '8':1}, inplace=True) #counts 429
df_2021['동영상_연예'].replace({' ':0, '9':1}, inplace=True) #counts 577
df_2021['동영상_스포츠'].replace({' ':0, '10':1}, inplace=True) #counts 321
df_2021['동영상_지역정보'].replace({' ':0, '11':1}, inplace=True) #counts 57
df_2021['동영상_사설'].replace({' ':0, '12':1}, inplace=True) #counts 66

df_2021['동영상주제수'] = df_2021[['동영상_국제', '동영상_정치', '동영상_사회', '동영상_경제', '동영상_교육', '동영상_과학',
                        '동영상_남북', '동영상_생활', '동영상_연예', '동영상_스포츠', '동영상_지역정보', '동영상_사설']].sum(axis=1)

df_2021.rename(columns={'Q70_1':'동영상뉴스활동_1', 'Q70_2':'동영상뉴스활동_2', 'Q70_3':'동영상뉴스활동_3'}, inplace=True) # 1-공유, 2-공감표시, 3-댓글
df_2021['동영상뉴스활동_1'].replace({2:0}, inplace=True) #없다 = 0, 있다 = 1/ counts 210
df_2021['동영상뉴스활동_2'].replace({2:0}, inplace=True) #counts 310
df_2021['동영상뉴스활동_3'].replace({2:0}, inplace=True) #counts 180
df_2021['동영상뉴스활동'] = df_2021[['동영상뉴스활동_1', '동영상뉴스활동_2', '동영상뉴스활동_3']].sum(axis=1) #mean=0.14
df_2021.loc[(df_2021['세대'] == 'old') | (df_2021['세대'] == 'young'), '동영상뉴스활동'].mean()

#t-test 동영상 이용량
ttest_left = df_2021.loc[df_2021['세대']=='young', '동영상이용일수']
ttest_right = df_2021.loc[df_2021['세대']=='old', '동영상이용일수']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

#t-test 동영상 뉴스 주제
ttest_left = df_2021.loc[df_2021['세대']=='young', '동영상_국제']
ttest_right = df_2021.loc[df_2021['세대']=='old', '동영상_국제']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '동영상_정치']
ttest_right = df_2021.loc[df_2021['세대']=='old', '동영상_정치']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '동영상_사회']
ttest_right = df_2021.loc[df_2021['세대']=='old', '동영상_사회']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '동영상_경제']
ttest_right = df_2021.loc[df_2021['세대']=='old', '동영상_경제']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '동영상_교육']
ttest_right = df_2021.loc[df_2021['세대']=='old', '동영상_교육']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '동영상_과학']
ttest_right = df_2021.loc[df_2021['세대']=='old', '동영상_과학']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '동영상_남북']
ttest_right = df_2021.loc[df_2021['세대']=='old', '동영상_남북']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '동영상_생활']
ttest_right = df_2021.loc[df_2021['세대']=='old', '동영상_생활']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '동영상_연예']
ttest_right = df_2021.loc[df_2021['세대']=='old', '동영상_연예']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '동영상_스포츠']
ttest_right = df_2021.loc[df_2021['세대']=='old', '동영상_스포츠']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '동영상_지역정보']
ttest_right = df_2021.loc[df_2021['세대']=='old', '동영상_지역정보']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '동영상_사설']
ttest_right = df_2021.loc[df_2021['세대']=='old', '동영상_사설']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest)

ttest_left = df_2021.loc[df_2021['세대']=='young', '동영상주제수']
ttest_right = df_2021.loc[df_2021['세대']=='old', '동영상주제수']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)

#t-test 동영상 뉴스 활동
ttest_left = df_2021.loc[df_2021['세대']=='young', '동영상뉴스활동']
ttest_right = df_2021.loc[df_2021['세대']=='old', '동영상뉴스활동']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #분산 동일성 검정 p = 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest)



#언론과 언론인에 대한 평가 (IV)
evaluation = df_2021[['Q78_1', 'Q78_2', 'Q78_3']]
pg.cronbach_alpha(data=evaluation) #신뢰도 측정 (cronbach alpha) a = .80
df_2021['언론평가'] = df_2021[['Q78_1', 'Q78_2', 'Q78_3']].mean(axis=1) #mean=3.25

#사회참여
df_2021['Q96_1'].replace({2:0}, inplace=True) #서명운동 557
df_2021['Q96_2'].replace({2:0}, inplace=True) #기부 771
df_2021['Q96_3'].replace({2:0}, inplace=True) #봉사 429
df_2021['Q96_4'].replace({2:0}, inplace=True) #의견표명 361

df_2021['사회참여'] = df_2021[['Q96_1', 'Q96_2', 'Q96_3', 'Q96_4']].sum(axis=1) #mean=0.42

df_2021.rename(columns={'BQ3':'학력', 'BQ5':'소득', 'BQ7':'정치성향', 'BQ8':'정치사회관심'}, inplace=True)

df = df_2021[['거주지역', '도시규모', '성별', '연령', '뉴스이용미디어', '동영상이용여부', '동영상뉴스이용여부', '동영상이용일수', '동영상뉴스이용일수',
              '동영상_국제', '동영상_정치', '동영상_사회', '동영상_경제', '동영상_교육', '동영상_과학', '동영상_남북',
              '동영상_생활', '동영상_연예', '동영상_스포츠', '동영상_지역정보', '동영상_사설', '언론평가', '사회참여', '학력', '소득', '정치성향', '정치사회관심']]

df.to_csv('2021_data.csv')
```