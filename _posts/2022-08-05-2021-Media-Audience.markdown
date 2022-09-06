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
df_2021.loc[df_2021['연령'] <= 26,'세대']='Z'
df_2021.loc[(df_2021['연령'] >= 58) & (df_2021['연령'] <= 66), '세대']='BB'
```

pandas 모듈을 이용하여 csv 파일을 불러올 수 있다. csv 파일의 변수 이름과 값을 사용하기 쉽게 변경하는 과정을 거치는 것이 좋다. 전체 코드 중 일부를 가져왔다.

데이터프레임.rename(columns={'기존 이름':'새로운 이름'}, inplace=True)은 열 이름을 바꿔주는 명령어다.

데이터프레임['변수 이름'].replace({'기존 값':'바꿔줄 값'}, inplace=True)는 해당 변수의 값을 변경하는 명령어다.

세대 구분을 위해 연령 변수를 이용하여 세대 변수를 생성하였다. Z세대는 2021년을 기준으로 26세 이하로 설정하였고, Baby Boomer세대는 58세에서 66세로 설정하였다. 변수의 값을 변경하거나 새로 생성할 때, 조건을 추가하려면 위와 같은 방법으로 실행하면 된다.

---

#### 디지털 미디어 이용

Z세대와 베이비 붐 세대의 디지털 미디어 이용 행태를 살펴보기 위해 가장 먼저 디지털 미디어 이용량을 살펴봤다.

![Alt text](/images/q5.jpg)
![Alt text](/images/q6.jpg)

```python 
df_2021['포털이용일수'] = df_2021[['Q38x1_1', 'Q38x2_1']].sum(axis=1)
df_2021['메신저이용일수'] = df_2021[['Q48x1', 'Q48x2']].sum(axis=1)
df_2021['SNS이용일수'] = df_2021[['Q56x1', 'Q56x2']].sum(axis=1)
df_2021['동영상이용일수'] = df_2021[['Q64x1', 'Q64x2']].sum(axis=1)
```

미디어 이용에 관한 문항은 '1주일 중 해당 미디어를 몇 일 이용했는가'로 측정되었다. 평일과 주말이 나누어 측정되어 있어서, 값을 합치는 과정을 거쳤다. .sum(axis=1)을 실행하면 다른 열의 값과 합할 수 있다. 이를 토대로 7점 척도의 디지털 미디어(포털, 메신저, SNS, 동영상) 이용 변수를 생성하였다.

```python
#t-test 포털 이용량
ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털이용일수'] #mean = 6.34
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털이용일수'] #mean = 4.01
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 18.96, p = 0.00 ***

#t-test 메신저 이용량
ttest_left = df_2021.loc[df_2021['세대']=='Z', '메신저이용일수'] #mean = 6.56
ttest_right = df_2021.loc[df_2021['세대']=='BB', '메신저이용일수'] #mean = 4.68
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t= 15.42, p = 0.00 ***

#t-test SNS 이용량
ttest_left = df_2021.loc[df_2021['세대']=='Z', 'SNS이용일수']
ttest_right = df_2021.loc[df_2021['세대']=='BB', 'SNS이용일수']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 27.39, p = 0.00 ***

#t-test 동영상 이용량
ttest_left = df_2021.loc[df_2021['세대']=='Z', '동영상이용일수'] #mean = 5.45
ttest_right = df_2021.loc[df_2021['세대']=='BB', '동영상이용일수'] #mean = 2.90
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 17.27, p = 0.00 ***
```

두 세대간 디지털 미디어 이용량에 유의미한 차이가 있는지 확인하기 위하여 독립표본 t검정을 실시하였다. t검정은 지난 언론인의 성 불평등 문제 인식 프로젝트에서도 다룬 적이 있다. 포털, 메신저, SNS, 동영상 미디어 모두 Z세대가 Baby Boomer세대보다 통계적으로 유의미하게 더 많이 이용하는 것을 확인하였다.

![Alt text](/images/g8.jpg)
![Alt text](/images/t8.jpg)

해당 결과를 표와 그래프로 정리하였다. (*논문 작성을 위해 표와 그래프는 엑셀로 제작하였다) Z세대와 Baby Boomer세대 모두 메신저의 이용량이 가장 많으며, 포털, 동영상, SNS 순으로 같은 패턴을 보이고 있다. 두 세대간 차이가 가장 큰 미디어는 SNS인 점을 확인할 수 있다.


---

#### 디지털 미디어 뉴스 참여

Z세대와 베이비 붐 세대의 디지털 미디어 이용 행태를 살펴보기 위해 뉴스 참여 수준을 살펴봤다.

![Alt text](/images/q7.jpg)
![Alt text](/images/q8.jpg)

```python
import pingouin as pg #Cronbach's alpha

#포털
df_2021.rename(columns={'Q44_1':'포털뉴스활동_1', 'Q44_2':'포털뉴스활동_2', 'Q44_3':'포털뉴스활동_3'}, inplace=True) # 1-공유, 2-공감표시, 3-댓글
df_2021['포털뉴스활동_1'].replace({2:0}, inplace=True)
df_2021['포털뉴스활동_2'].replace({2:0}, inplace=True)
df_2021['포털뉴스활동_3'].replace({2:0}, inplace=True)

evaluation = df_2021[['포털뉴스활동_1', '포털뉴스활동_2', '포털뉴스활동_3']]
pg.cronbach_alpha(data=evaluation) #a = 0.68
df_2021['포털뉴스활동'] = df_2021[['포털뉴스활동_1', '포털뉴스활동_2', '포털뉴스활동_3']].sum(axis=1)

#메신저
df_2021.rename(columns={'Q54_1':'메신저뉴스활동_1', 'Q54_2':'메신저뉴스활동_2', 'Q54_3':'메신저뉴스활동_3'}, inplace=True)
df_2021['메신저뉴스활동_1'].replace({2:0}, inplace=True)
df_2021['메신저뉴스활동_2'].replace({2:0}, inplace=True)
df_2021['메신저뉴스활동_3'].replace({2:0}, inplace=True)

evaluation = df_2021[['메신저뉴스활동_1', '메신저뉴스활동_2', '메신저뉴스활동_3']]
pg.cronbach_alpha(data=evaluation)  #a = 0.79
df_2021['메신저뉴스활동'] = df_2021[['메신저뉴스활동_1', '메신저뉴스활동_2', '메신저뉴스활동_3']].sum(axis=1)

#SNS
df_2021.rename(columns={'Q62_1':'SNS뉴스활동_1', 'Q62_2':'SNS뉴스활동_2', 'Q62_3':'SNS뉴스활동_3'}, inplace=True) # 1-공유, 2-공감표시, 3-댓글
df_2021['SNS뉴스활동_1'].replace({2:0}, inplace=True)
df_2021['SNS뉴스활동_2'].replace({2:0}, inplace=True)
df_2021['SNS뉴스활동_3'].replace({2:0}, inplace=True)

evaluation = df_2021[['SNS뉴스활동_1', 'SNS뉴스활동_2', 'SNS뉴스활동_3']]
pg.cronbach_alpha(data=evaluation) #a = 0.85
df_2021['SNS뉴스활동'] = df_2021[['SNS뉴스활동_1', 'SNS뉴스활동_2', 'SNS뉴스활동_3']].sum(axis=1)

#동영상
df_2021.rename(columns={'Q70_1':'동영상뉴스활동_1', 'Q70_2':'동영상뉴스활동_2', 'Q70_3':'동영상뉴스활동_3'}, inplace=True)
df_2021['동영상뉴스활동_1'].replace({2:0}, inplace=True)
df_2021['동영상뉴스활동_2'].replace({2:0}, inplace=True)
df_2021['동영상뉴스활동_3'].replace({2:0}, inplace=True)

evaluation = df_2021[['동영상뉴스활동_1', '동영상뉴스활동_2', '동영상뉴스활동_3']]
pg.cronbach_alpha(data=evaluation) #a = 0.74
df_2021['동영상뉴스활동'] = df_2021[['동영상뉴스활동_1', '동영상뉴스활동_2', '동영상뉴스활동_3']].sum(axis=1)
```

뉴스 참여는 위 설문 문항과 같이 공유, 공감표시, 댓글 이 세가지 활동의 여부를 묻는 형식으로 진행되었다. 일단 [1=있다, 2=없다]로 측정되어있는 값을 [0=없다, 1=있다]로 변경하는 과정을 거쳤다. 이후 세 문항의 크론바흐 알파계수를 확인하였고, 0~3점 척도의 뉴스 참여 변수를 새로 생성하였다.

```python 
#t-test 포털 참여
ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털뉴스활동'] #mean = 0.45
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털뉴스활동'] #mean = 0.07
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 9.25, p = 0.00 ***

#t-test 메신저 참여
ttest_left = df_2021.loc[df_2021['세대']=='Z', '메신저뉴스활동'] #mean = 0.19
ttest_right = df_2021.loc[df_2021['세대']=='BB', '메신저뉴스활동'] #mean = 0.03
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 5.41, p = 0.00 ***

#t-test SNS 참여
ttest_left = df_2021.loc[df_2021['세대']=='Z', 'SNS뉴스활동'] #mean = 0.25
ttest_right = df_2021.loc[df_2021['세대']=='BB', 'SNS뉴스활동'] #mean = 0.03
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 6.58, p = 0.00 ***

#t-test 동영상 참여
ttest_left = df_2021.loc[df_2021['세대']=='Z', '동영상뉴스활동'] #mean = 0.19
ttest_right = df_2021.loc[df_2021['세대']=='BB', '동영상뉴스활동'] #mean = 0.06
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 4.31, p = 0.00
```

두 세대간 디지털 미디어 뉴스 참여 수준에 유의미한 차이가 있는지 확인하기 위하여 미디어 이용량과 마찬가지로 t-test 분석을 실시하였다. Z세대가 Baby Boomer세대보다 통계적으로 유의미하게 뉴스 참여를 더 많이 하는 것으로 나타났다.

![Alt text](/images/g9.jpg)
![Alt text](/images/t9.jpg)

위 분석 결과를 표와 그래프로 정리하였다. 세대별로 보면 Z세대의 경우 인터넷 포털에서 가장 많이 뉴스 참여를 하였으며, 다음으로 SNS, 동영상 메신저 순 이었다. Baby Boomer세대는 Z세대와 같이 포털에서의 뉴스 참여가 가장 많았지만 다음으로 동영상, 메신저, SNS 순으로 다른 패턴을 보였다.

---

#### 디지털 미디어 뉴스 주제 이용

마지막으로, Z세대와 베이비 붐 세대의 디지털 미디어 이용 행태를 살펴보기 위해 디지털 미디어 뉴스 주제 이용을 살펴봤다.

![Alt text](/images/q9.jpg)

뉴스 주제 이용은 위 설문 문항과 같이 12개의 범주로 측정되었다(기타 제외). 복수 응답이 가능하게 하여 각 주제마다 결측값과 특정 값이 입력되어 있었다. 예를 들어 국제와 경제에만 체크했을 경우, 국제와 경제는 1,4로 입력되고 나머지 주제는 결측 값이 입력되었다.

```python 
df_2021.rename(columns={'Q43_1':'포털_국제', 'Q43_2':'포털_정치', 'Q43_3':'포털_사회', 'Q43_4':'포털_경제', 'Q43_5':'포털_교육', 'Q43_6':'포털_과학',
                        'Q43_7':'포털_남북', 'Q43_8':'포털_생활', 'Q43_9':'포털_연예', 'Q43_10':'포털_스포츠', 'Q43_11':'포털_지역정보', 'Q43_12':'포털_사설'}, inplace=True)
df_2021['포털_국제'].replace({' ':0, '1':1}, inplace=True)
df_2021['포털_정치'].replace({' ':0, '2':1}, inplace=True)
df_2021['포털_사회'].replace({' ':0, '3':1}, inplace=True)
df_2021['포털_경제'].replace({' ':0, '4':1}, inplace=True)
df_2021['포털_교육'].replace({' ':0, '5':1}, inplace=True)
df_2021['포털_과학'].replace({' ':0, '6':1}, inplace=True)
df_2021['포털_남북'].replace({' ':0, '7':1}, inplace=True)
df_2021['포털_생활'].replace({' ':0, '8':1}, inplace=True)
df_2021['포털_연예'].replace({' ':0, '9':1}, inplace=True)
df_2021['포털_스포츠'].replace({' ':0, '10':1}, inplace=True)
df_2021['포털_지역정보'].replace({' ':0, '11':1}, inplace=True)
df_2021['포털_사설'].replace({' ':0, '12':1}, inplace=True)

#t-test 포털 뉴스 주제
ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_국제'] #mean = 0.27
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_국제'] #mean = 0.21
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.01
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 2.39, p = 0.02 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_정치'] #mean = 0.41
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_정치'] #mean = 0.45
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.15
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = -1.44, p = 0.15

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_사회'] #mean = 0.76
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_사회'] #mean = 0.56
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 7.62, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_경제'] #mean = 0.51
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_경제'] #mean = 0.46
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.28
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 1.60, p = 0.11

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_교육'] #mean = 0.26
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_교육'] #mean = 0.11
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 6.46, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_과학'] #mean = 0.18
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_과학'] #mean = 0.10
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 4.17, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_남북'] #mean = 0.09
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_남북'] #mean = 0.11
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.44
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = -0.78, p = 0.44

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_생활'] #mean = 0.50
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_생활'] #mean = 0.37
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 4.38, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_연예'] #mean = 0.67
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_연예'] #mean = 0.24
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 16.33, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_스포츠'] #mean = 0.33
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_스포츠'] #mean = 0.19
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 5.46, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_지역정보'] #mean = 0.07
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_지역정보'] #mean = 0.12
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.01
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = -2.85, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_사설'] #mean = 0.05
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_사설'] #mean = 0.06
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.68
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = -0.41, p = 0.68
```

위 코드와 같이, 응답한 항목에 대하여 변수의 값을 0,1로 변경하였다. 즉 응답한 경우 1로 입력하였고 응답하지 않은 항목의 경우 0으로 입력하였다. 이 과정을 거친 후, 각 주제마다 세대간 t-test분석을 실시하였다. 포털 이외의 미디어 t-test 코드는 전체 코드에 같이 첨부해 두었다.

![Alt text](/images/g10.jpg)
![Alt text](/images/g11.jpg)

포털, 메신저, SNS, 동영상 미디어의 뉴스 주제 이용을 모두 살펴본 결과를 다음과 같은 그래프로 나타냈다. 

Z세대가 주로 보는 뉴스의 주제 중 가장 많이 응답한 주제는 '사회'이다. 다음으로는 연예, 경제, 생활, 정치가 상위에 있다. 모든 주제를 인터넷 포털 사이트에서 가장 많이 이용한 것을 볼 수 있다. 
Baby Boomer세대가 주로 보는 뉴스의 주제 중 가장 많이 응답한 주제는 Z세대와 같은 '사회'이다. 다음으로는 정치, 경제, 생활, 연예 순이다.
두 집단 모두 사회 주제를 가장 많이 이용하는 점은 같았지만, Z세대는 주로 연예 주제를 경제, 정치보다 많이 이용한 반면, Baby Boomer세대는 정치, 경제 주제를 주로 이용하는 것으로 나타났다.

![Alt text](/images/t10.jpg)

두 집단의 뉴스 주제 이용의 t-test 결과를 정리한 표이다. 
포털에서는 국제, 사회, 교육, 과학, 생활, 연예, 스포츠 주제에서 Z세대가 베이비붐 세대보다 유의미하게 많이 응답한 것으로 나타났으며, 지역정보의 경우에는 베이비붐 세대가 Z세대보다 통계적으로 유의미하게 많이 이용한 점을 발견했다. 
메신저의 경우, 국제, 사회, 경제, 교육, 생활, 연예, 스포츠 주제를 Z세대가 베이비붐 세대보다 많이 이용했다는 점을 볼 수 있다. SNS의 경우, 사설을 제외한 주제에서 유의미한 차이를 보였다. 동영상의 경우에는 사회, 교육, 과학, 생활, 연예, 스포츠 주제에서 유의미한 차이를 보였으며, 4개의 디지털 미디어 중 유일하게 국제 주제에서 유의미한 차이가 발견되지 않았다. 

---

이번 프로젝트는 [Z세대와 베이비 붐 세대의 디지털 미디어 이용 행태]로 미디어 수용자들의 디지털 미디어 이용 행태를 Z세대와 Baby Boomer세대 중심으로 살펴봤다. 이 분석 결과를 통해 논문을 작성하여 2021 통계청 논문 공모전에 제출하였다. 
Z세대와 Baby Boomer세대의 디지털 미디어 이용이 어떻게 다른지 뿐만 아니라, 미디어를 통한 뉴스 참여의 수준과 이용하는 뉴스 주제의 차이까지 알아볼 수 있었다. 세대간 미디어 이용 패턴과 뉴스 주제 이용을 탐색한 것은 빠르게 변동하는 디지털 미디어 환경 속에서 세대간 갈등을 해소하는 중요한 지침이 되리라 생각한다.

한국언론재단이 제공하는 언론인 인식과, 언론 수용자 인식 조사를 통해 여러 프로젝트를 진행하였다. 하나의 데이터로도 여러 의미있는 결과를 도출할 수 있다는 점을 다시 한 번 깨달을 수 있었다. 또한 이번 기회로 논문에 연구방법과 연구결과를 기술하며, 분석 결과를 정리하는 법을 배울 수 있었다.
이번 프로젝트를 진행하면서는 특히 뉴스 주제 이용 변수를 다룰 때, 어려운 점이 많았다. 복수 응답으로 측정되어 교차분석은 불가능하였다. 변수 값 변경과 항목별 t-test분석은 분명히 더 효율적인 코드가 있을 것이라 생각한다. for문과 함수 생성이 조금 더 익숙해지면 다시 코드를 짜보는 것도 좋은 경험일 것 같다.

전체 코드는 본문과 순서가 다르다. 본문은 연구 주제에 맞게 순서를 재배치하였고, 전체 코드는 미디어별로 먼저 변수를 정리한 후, 분석을 실시하는 순서로 되어있다. 이 점 참고한 후에 전체 코드를 참고하길 바란다.

---

#### 전체 Code
```python 
import numpy as np
import pandas as pd
import pingouin as pg #Cronbach's alpha
from scipy import stats


df_2021 = pd.read_csv("2021 언론수용자 조사 DATA_공개용_최종.csv")

#인구통계학
df_2021.rename(columns={'SQ1':'거주지역','SQ2':'도시규모','SQ3':'성별','SQ4':'연령'}, inplace=True)
df_2021['거주지역'].replace({1:1, 2:0, 3:0, 4:0, 5:0, 6:0, 7:0, 8:0, 9:0, 10:0, 11:0, 12:0, 13:0, 14:0, 15:0, 16:0, 17:0}, inplace=True) #서울 1 나머지 0
df_2021['성별'].replace({1:0, 2:1}, inplace=True) # 0 = 남성 (2488) , 1 = 여성 (2522)

df_2021['세대'] = np.nan
df_2021.loc[df_2021['연령'] <= 26,'세대']='Z'
df_2021.loc[(df_2021['연령'] >= 58) & (df_2021['연령'] <= 66), '세대']='BB'


#인터넷 포털 및 언론사 사이트 이용
df_2021['포털이용일수'] = df_2021[['Q38x1_1', 'Q38x2_1']].sum(axis=1)

df_2021.rename(columns={'Q43_1':'포털_국제', 'Q43_2':'포털_정치', 'Q43_3':'포털_사회', 'Q43_4':'포털_경제', 'Q43_5':'포털_교육', 'Q43_6':'포털_과학',
                        'Q43_7':'포털_남북', 'Q43_8':'포털_생활', 'Q43_9':'포털_연예', 'Q43_10':'포털_스포츠', 'Q43_11':'포털_지역정보', 'Q43_12':'포털_사설'}, inplace=True)
df_2021['포털_국제'].replace({' ':0, '1':1}, inplace=True)
df_2021['포털_정치'].replace({' ':0, '2':1}, inplace=True)
df_2021['포털_사회'].replace({' ':0, '3':1}, inplace=True)
df_2021['포털_경제'].replace({' ':0, '4':1}, inplace=True)
df_2021['포털_교육'].replace({' ':0, '5':1}, inplace=True)
df_2021['포털_과학'].replace({' ':0, '6':1}, inplace=True)
df_2021['포털_남북'].replace({' ':0, '7':1}, inplace=True)
df_2021['포털_생활'].replace({' ':0, '8':1}, inplace=True)
df_2021['포털_연예'].replace({' ':0, '9':1}, inplace=True)
df_2021['포털_스포츠'].replace({' ':0, '10':1}, inplace=True)
df_2021['포털_지역정보'].replace({' ':0, '11':1}, inplace=True)
df_2021['포털_사설'].replace({' ':0, '12':1}, inplace=True)

df_2021.rename(columns={'Q44_1':'포털뉴스활동_1', 'Q44_2':'포털뉴스활동_2', 'Q44_3':'포털뉴스활동_3'}, inplace=True) # 1-공유, 2-공감표시, 3-댓글
df_2021['포털뉴스활동_1'].replace({2:0}, inplace=True)
df_2021['포털뉴스활동_2'].replace({2:0}, inplace=True)
df_2021['포털뉴스활동_3'].replace({2:0}, inplace=True)

evaluation = df_2021[['포털뉴스활동_1', '포털뉴스활동_2', '포털뉴스활동_3']]
pg.cronbach_alpha(data=evaluation) #a = 0.68
df_2021['포털뉴스활동'] = df_2021[['포털뉴스활동_1', '포털뉴스활동_2', '포털뉴스활동_3']].sum(axis=1)

#t-test 포털 이용량
ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털이용일수'] #mean = 6.34
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털이용일수'] #mean = 4.01
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 18.96, p = 0.00 ***

#t-test 포털 뉴스 주제
ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_국제'] #mean = 0.27
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_국제'] #mean = 0.21
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.01
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 2.39, p = 0.02 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_정치'] #mean = 0.41
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_정치'] #mean = 0.45
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.15
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = -1.44, p = 0.15

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_사회'] #mean = 0.76
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_사회'] #mean = 0.56
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 7.62, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_경제'] #mean = 0.51
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_경제'] #mean = 0.46
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.28
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 1.60, p = 0.11

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_교육'] #mean = 0.26
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_교육'] #mean = 0.11
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 6.46, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_과학'] #mean = 0.18
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_과학'] #mean = 0.10
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 4.17, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_남북'] #mean = 0.09
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_남북'] #mean = 0.11
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.44
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = -0.78, p = 0.44

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_생활'] #mean = 0.50
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_생활'] #mean = 0.37
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 4.38, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_연예'] #mean = 0.67
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_연예'] #mean = 0.24
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 16.33, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_스포츠'] #mean = 0.33
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_스포츠'] #mean = 0.19
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 5.46, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_지역정보'] #mean = 0.07
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_지역정보'] #mean = 0.12
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.01
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = -2.85, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털_사설'] #mean = 0.05
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털_사설'] #mean = 0.06
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.68
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = -0.41, p = 0.68

#t-test 포털 미디어 참여
ttest_left = df_2021.loc[df_2021['세대']=='Z', '포털뉴스활동'] #mean = 0.45
ttest_right = df_2021.loc[df_2021['세대']=='BB', '포털뉴스활동'] #mean = 0.07
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 9.25, p = 0.00 ***


#메신저 서비스 이용
df_2021['메신저이용일수'] = df_2021[['Q48x1', 'Q48x2']].sum(axis=1)

df_2021.rename(columns={'Q53_1':'메신저_국제', 'Q53_2':'메신저_정치', 'Q53_3':'메신저_사회', 'Q53_4':'메신저_경제', 'Q53_5':'메신저_교육', 'Q53_6':'메신저_과학',
                        'Q53_7':'메신저_남북', 'Q53_8':'메신저_생활', 'Q53_9':'메신저_연예', 'Q53_10':'메신저_스포츠', 'Q53_11':'메신저_지역정보', 'Q53_12':'메신저_사설'}, inplace=True)
df_2021['메신저_국제'].replace({' ':0, '1':1}, inplace=True)
df_2021['메신저_정치'].replace({' ':0, '2':1}, inplace=True)
df_2021['메신저_사회'].replace({' ':0, '3':1}, inplace=True)
df_2021['메신저_경제'].replace({' ':0, '4':1}, inplace=True)
df_2021['메신저_교육'].replace({' ':0, '5':1}, inplace=True)
df_2021['메신저_과학'].replace({' ':0, '6':1}, inplace=True)
df_2021['메신저_남북'].replace({' ':0, '7':1}, inplace=True)
df_2021['메신저_생활'].replace({' ':0, '8':1}, inplace=True)
df_2021['메신저_연예'].replace({' ':0, '9':1}, inplace=True)
df_2021['메신저_스포츠'].replace({' ':0, '10':1}, inplace=True)
df_2021['메신저_지역정보'].replace({' ':0, '11':1}, inplace=True)
df_2021['메신저_사설'].replace({' ':0, '12':1}, inplace=True)

df_2021.rename(columns={'Q54_1':'메신저뉴스활동_1', 'Q54_2':'메신저뉴스활동_2', 'Q54_3':'메신저뉴스활동_3'}, inplace=True)
df_2021['메신저뉴스활동_1'].replace({2:0}, inplace=True)
df_2021['메신저뉴스활동_2'].replace({2:0}, inplace=True)
df_2021['메신저뉴스활동_3'].replace({2:0}, inplace=True)

evaluation = df_2021[['메신저뉴스활동_1', '메신저뉴스활동_2', '메신저뉴스활동_3']]
pg.cronbach_alpha(data=evaluation)  #a=0.79
df_2021['메신저뉴스활동'] = df_2021[['메신저뉴스활동_1', '메신저뉴스활동_2', '메신저뉴스활동_3']].sum(axis=1)

#t-test 메신저 이용량
ttest_left = df_2021.loc[df_2021['세대']=='Z', '메신저이용일수'] #mean = 6.56
ttest_right = df_2021.loc[df_2021['세대']=='BB', '메신저이용일수'] #mean = 4.68
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t= 15.42, p = 0.00 ***

#t-test 메신저 뉴스 주제
ttest_left = df_2021.loc[df_2021['세대']=='Z', '메신저_국제'] #mean = 0.03
ttest_right = df_2021.loc[df_2021['세대']=='BB', '메신저_국제'] #mean = 0.01
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.01
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 2.30, p = 0.02 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '메신저_정치'] #mean = 0.05
ttest_right = df_2021.loc[df_2021['세대']=='BB', '메신저_정치'] #mean = 0.04
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.35
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 0.93, p = 0.35

ttest_left = df_2021.loc[df_2021['세대']=='Z', '메신저_사회'] #mean = 0.18
ttest_right = df_2021.loc[df_2021['세대']=='BB', '메신저_사회'] #mean = 0.07
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 5.64, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '메신저_경제'] #mean = 0.09
ttest_right = df_2021.loc[df_2021['세대']=='BB', '메신저_경제'] #mean = 0.04
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) # 0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 3.43, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '메신저_교육'] #mean = 0.04
ttest_right = df_2021.loc[df_2021['세대']=='BB', '메신저_교육'] #mean = 0.01
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 2.56, p = 0.01 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '메신저_과학'] #mean = 0.02
ttest_right = df_2021.loc[df_2021['세대']=='BB', '메신저_과학'] #mean = 0.01
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.05
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 1.93, p = 0.05

ttest_left = df_2021.loc[df_2021['세대']=='Z', '메신저_남북'] #mean = 0.02
ttest_right = df_2021.loc[df_2021['세대']=='BB', '메신저_남북'] #mean = 0.01
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.07
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 1.83, p = 0.07

ttest_left = df_2021.loc[df_2021['세대']=='Z', '메신저_생활'] #mean = 0.08
ttest_right = df_2021.loc[df_2021['세대']=='BB', '메신저_생활'] #mean = 0.03
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 3.50, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '메신저_연예'] #mean = 0.17
ttest_right = df_2021.loc[df_2021['세대']=='BB', '메신저_연예'] #mean = 0.03
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 7.74, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '메신저_스포츠'] #mean = 0.06
ttest_right = df_2021.loc[df_2021['세대']=='BB', '메신저_스포츠'] #mean = 0.01
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 4.21, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '메신저_지역정보'] #mean = 0.01
ttest_right = df_2021.loc[df_2021['세대']=='BB', '메신저_지역정보'] #mean = 0.01
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.26
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 1.13, p = 0.26

ttest_left = df_2021.loc[df_2021['세대']=='Z', '메신저_사설'] #mean = 0.00
ttest_right = df_2021.loc[df_2021['세대']=='BB', '메신저_사설'] #mean = 0.00
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.35
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = -0.93, p =0.35

#t-test 메신저 참여
ttest_left = df_2021.loc[df_2021['세대']=='Z', '메신저뉴스활동'] #mean = 0.19
ttest_right = df_2021.loc[df_2021['세대']=='BB', '메신저뉴스활동'] #mean = 0.03
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 5.41, p = 0.00 ***

#SNS 이용
df_2021['SNS이용일수'] = df_2021[['Q56x1', 'Q56x2']].sum(axis=1)

df_2021.rename(columns={'Q61_1':'SNS_국제', 'Q61_2':'SNS_정치', 'Q61_3':'SNS_사회', 'Q61_4':'SNS_경제', 'Q61_5':'SNS_교육', 'Q61_6':'SNS_과학',
                        'Q61_7':'SNS_남북', 'Q61_8':'SNS_생활', 'Q61_9':'SNS_연예', 'Q61_10':'SNS_스포츠', 'Q61_11':'SNS_지역정보', 'Q61_12':'SNS_사설'}, inplace=True)
df_2021['SNS_국제'].replace({' ':0, '1':1}, inplace=True)
df_2021['SNS_정치'].replace({' ':0, '2':1}, inplace=True)
df_2021['SNS_사회'].replace({' ':0, '3':1}, inplace=True)
df_2021['SNS_경제'].replace({' ':0, '4':1}, inplace=True)
df_2021['SNS_교육'].replace({' ':0, '5':1}, inplace=True)
df_2021['SNS_과학'].replace({' ':0, '6':1}, inplace=True)
df_2021['SNS_남북'].replace({' ':0, '7':1}, inplace=True)
df_2021['SNS_생활'].replace({' ':0, '8':1}, inplace=True)
df_2021['SNS_연예'].replace({' ':0, '9':1}, inplace=True)
df_2021['SNS_스포츠'].replace({' ':0, '10':1}, inplace=True)
df_2021['SNS_지역정보'].replace({' ':0, '11':1}, inplace=True)
df_2021['SNS_사설'].replace({' ':0, '12':1}, inplace=True)

df_2021.rename(columns={'Q62_1':'SNS뉴스활동_1', 'Q62_2':'SNS뉴스활동_2', 'Q62_3':'SNS뉴스활동_3'}, inplace=True) # 1-공유, 2-공감표시, 3-댓글
df_2021['SNS뉴스활동_1'].replace({2:0}, inplace=True)
df_2021['SNS뉴스활동_2'].replace({2:0}, inplace=True)
df_2021['SNS뉴스활동_3'].replace({2:0}, inplace=True)

evaluation = df_2021[['SNS뉴스활동_1', 'SNS뉴스활동_2', 'SNS뉴스활동_3']]
pg.cronbach_alpha(data=evaluation) #a=0.85
df_2021['SNS뉴스활동'] = df_2021[['SNS뉴스활동_1', 'SNS뉴스활동_2', 'SNS뉴스활동_3']].sum(axis=1)

#t-test SNS 이용량
ttest_left = df_2021.loc[df_2021['세대']=='Z', 'SNS이용일수']
ttest_right = df_2021.loc[df_2021['세대']=='BB', 'SNS이용일수']
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 27.39, p = 0.00 ***

#t-test SNS 뉴스 주제
ttest_left = df_2021.loc[df_2021['세대']=='Z', 'SNS_국제'] #mean = 0.05
ttest_right = df_2021.loc[df_2021['세대']=='BB', 'SNS_국제'] #mean = 0.01
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 3.96, p = 0.00

ttest_left = df_2021.loc[df_2021['세대']=='Z', 'SNS_정치'] #mean = 0.08
ttest_right = df_2021.loc[df_2021['세대']=='BB', 'SNS_정치'] #mean = 0.01
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 5.22, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', 'SNS_사회'] #mean = 0.23
ttest_right = df_2021.loc[df_2021['세대']=='BB', 'SNS_사회'] #mean = 0.01
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 10.66, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', 'SNS_경제'] #mean = 0.10
ttest_right = df_2021.loc[df_2021['세대']=='BB', 'SNS_경제'] #mean = 0.01
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 6.22, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', 'SNS_교육'] #mean = 0.05
ttest_right = df_2021.loc[df_2021['세대']=='BB', 'SNS_교육'] #mean = 0.01
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 4.09, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', 'SNS_과학'] #mean = 0.03
ttest_right = df_2021.loc[df_2021['세대']=='BB', 'SNS_과학'] #mean = 0.00
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 3.90, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', 'SNS_남북'] #mean = 0.03
ttest_right = df_2021.loc[df_2021['세대']=='BB', 'SNS_남북'] #mean = 0.00
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 3.42, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', 'SNS_생활'] #mean = 0.10
ttest_right = df_2021.loc[df_2021['세대']=='BB', 'SNS_생활'] #mean = 0.01
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 6.40, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', 'SNS_연예'] #mean = 0.20
ttest_right = df_2021.loc[df_2021['세대']=='BB', 'SNS_연예'] #mean = 0.00
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 10.62, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', 'SNS_스포츠'] #mean = 0.07
ttest_right = df_2021.loc[df_2021['세대']=='BB', 'SNS_스포츠'] #mean = 0.00
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 5.93, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', 'SNS_지역정보'] #mean = 0.02
ttest_right = df_2021.loc[df_2021['세대']=='BB', 'SNS_지역정보'] #mean = 0.00
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 2.62, p = 0.01 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', 'SNS_사설'] #mean = 0.01
ttest_right = df_2021.loc[df_2021['세대']=='BB', 'SNS_사설'] #mean = 0.00
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.03
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 1.68, p = 0.09

#t-test SNS 참여
ttest_left = df_2021.loc[df_2021['세대']=='Z', 'SNS뉴스활동'] #mean = 0.25
ttest_right = df_2021.loc[df_2021['세대']=='BB', 'SNS뉴스활동'] #mean = 0.03
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 6.58, p = 0.00 ***


#온라인 동영상 플랫폼
df_2021['동영상이용일수'] = df_2021[['Q64x1', 'Q64x2']].sum(axis=1)

df_2021.rename(columns={'Q69_1':'동영상_국제', 'Q69_2':'동영상_정치', 'Q69_3':'동영상_사회', 'Q69_4':'동영상_경제', 'Q69_5':'동영상_교육', 'Q69_6':'동영상_과학',
                        'Q69_7':'동영상_남북', 'Q69_8':'동영상_생활', 'Q69_9':'동영상_연예', 'Q69_10':'동영상_스포츠', 'Q69_11':'동영상_지역정보', 'Q69_12':'동영상_사설'}, inplace=True)
df_2021['동영상_국제'].replace({' ':0, '1':1}, inplace=True)
df_2021['동영상_정치'].replace({' ':0, '2':1}, inplace=True)
df_2021['동영상_사회'].replace({' ':0, '3':1}, inplace=True)
df_2021['동영상_경제'].replace({' ':0, '4':1}, inplace=True)
df_2021['동영상_교육'].replace({' ':0, '5':1}, inplace=True)
df_2021['동영상_과학'].replace({' ':0, '6':1}, inplace=True)
df_2021['동영상_남북'].replace({' ':0, '7':1}, inplace=True)
df_2021['동영상_생활'].replace({' ':0, '8':1}, inplace=True)
df_2021['동영상_연예'].replace({' ':0, '9':1}, inplace=True)
df_2021['동영상_스포츠'].replace({' ':0, '10':1}, inplace=True)
df_2021['동영상_지역정보'].replace({' ':0, '11':1}, inplace=True)
df_2021['동영상_사설'].replace({' ':0, '12':1}, inplace=True)

df_2021.rename(columns={'Q70_1':'동영상뉴스활동_1', 'Q70_2':'동영상뉴스활동_2', 'Q70_3':'동영상뉴스활동_3'}, inplace=True)
df_2021['동영상뉴스활동_1'].replace({2:0}, inplace=True)
df_2021['동영상뉴스활동_2'].replace({2:0}, inplace=True)
df_2021['동영상뉴스활동_3'].replace({2:0}, inplace=True)

evaluation = df_2021[['동영상뉴스활동_1', '동영상뉴스활동_2', '동영상뉴스활동_3']]
pg.cronbach_alpha(data=evaluation) #a = 0.74
df_2021['동영상뉴스활동'] = df_2021[['동영상뉴스활동_1', '동영상뉴스활동_2', '동영상뉴스활동_3']].sum(axis=1)

#t-test 동영상 이용량
ttest_left = df_2021.loc[df_2021['세대']=='Z', '동영상이용일수'] #mean = 5.45
ttest_right = df_2021.loc[df_2021['세대']=='BB', '동영상이용일수'] #mean = 2.90
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 17.27, p = 0.00 ***

#t-test 동영상 뉴스 주제
ttest_left = df_2021.loc[df_2021['세대']=='Z', '동영상_국제'] #mean = 0.06
ttest_right = df_2021.loc[df_2021['세대']=='BB', '동영상_국제'] #mean = 0.04
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.07
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 1.84, p = 0.07

ttest_left = df_2021.loc[df_2021['세대']=='Z', '동영상_정치'] #mean = 0.10
ttest_right = df_2021.loc[df_2021['세대']=='BB', '동영상_정치'] #mean = 0.13
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.14
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = -1.49, p = 0.14

ttest_left = df_2021.loc[df_2021['세대']=='Z', '동영상_사회'] #mean = 0.26
ttest_right = df_2021.loc[df_2021['세대']=='BB', '동영상_사회'] #mean = 0.16
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 4.36, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '동영상_경제'] #mean = 0.10
ttest_right = df_2021.loc[df_2021['세대']=='BB', '동영상_경제'] #mean = 0.10
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.89
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = -0.14, p = 0.89

ttest_left = df_2021.loc[df_2021['세대']=='Z', '동영상_교육'] #mean = 0.05
ttest_right = df_2021.loc[df_2021['세대']=='BB', '동영상_교육'] #mean = 0.01
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 4.22, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '동영상_과학'] #mean = 0.03
ttest_right = df_2021.loc[df_2021['세대']=='BB', '동영상_과학'] #mean = 0.01
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 3.09, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '동영상_남북'] #mean = 0.02
ttest_right = df_2021.loc[df_2021['세대']=='BB', '동영상_남북'] #mean = 0.01
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.17
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 1.38, p = 0.17

ttest_left = df_2021.loc[df_2021['세대']=='Z', '동영상_생활'] #mean = 0.09
ttest_right = df_2021.loc[df_2021['세대']=='BB', '동영상_생활'] #mean = 0.06
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.02
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 2.24, p = 0.03

ttest_left = df_2021.loc[df_2021['세대']=='Z', '동영상_연예'] #mean = 0.19
ttest_right = df_2021.loc[df_2021['세대']=='BB', '동영상_연예'] #mean = 0.05
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 6.98, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '동영상_스포츠'] #mean = 0.08
ttest_right = df_2021.loc[df_2021['세대']=='BB', '동영상_스포츠'] #mean = 0.04
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 3.05, p = 0.00 ***

ttest_left = df_2021.loc[df_2021['세대']=='Z', '동영상_지역정보'] #mean = 0.01
ttest_right = df_2021.loc[df_2021['세대']=='BB', '동영상_지역정보'] #mean = 0.01
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.51
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 0.65, p = 0.51

ttest_left = df_2021.loc[df_2021['세대']=='Z', '동영상_사설'] #mean = 0.01
ttest_right = df_2021.loc[df_2021['세대']=='BB', '동영상_사설'] #mean = 0.01
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.76
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=True)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = -0.31, p = 0.76

#t-test 동영상 뉴스 활동
ttest_left = df_2021.loc[df_2021['세대']=='Z', '동영상뉴스활동'] #mean = 0.19
ttest_right = df_2021.loc[df_2021['세대']=='BB', '동영상뉴스활동'] #mean = 0.06
levene =stats.levene(ttest_left, ttest_right)
print("leveneresult(statistic = %.2f, pvalue = %.2f" % levene) #0.00
ttest = stats.ttest_ind(ttest_left, ttest_right, equal_var=False)
print("t-value = %.2f, p-value = %.2f" % ttest) #t = 4.31, p = 0.00
```