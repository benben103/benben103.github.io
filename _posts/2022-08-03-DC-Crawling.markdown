---
layout: post
title: 디시인사이드(커뮤니티 사이트) 크롤링
image: 4.jpg
date: 2022-08-01 17:58:18 +0200
tags: [Project, Crawling]
categories: Project
---

이번에는 디시인사이드 게시글을 크롤링하는 방법을 다루고자 한다. 지역 혐오 표현이 어떤 맥락에서 주로 등장하는지 살펴보기 위하여 디시인사이드 게시글을 크롤링하기로 하였다.

---

![Alt text](/images/p4.jpg)

디시인사이드 사이트에서 특정 키워드를 검색하면 다음과 같은 화면을 볼 수 있다. 커뮤니티 유저들이 지역 혐오 표현을 어떻게 사용하는지 살펴보기 위해 게시물을 크롤링을 하고자 한다

해당 화면의 링크를 보면 https://search.dcinside.com/post/p/1/sort/accuracy/q/.EC.84.9C.EC.9A.B8.EA.B3.B5.ED.99.94.EA.B5.AD 이다.

1은 검색 결과의 페이지를 나타내며 마지막 부분은 검색어를 유니코드로 입력되어 있다.

```python
import urllib

search = input("검색어 입력: ")
uni_kor = urllib.parse.quote(search)
```

위 코드로 입력한 글자를 유니코드로 바꿀 수 있다. 예시 화면에서 검색한 '서울공화국'의 유니코드는 ".EC.84.9C.EC.9A.B8.EA.B3.B5.ED.99.94.EA.B5.AD" 으로 링크 마지막 부분과 같은 것을 볼 수 있다.

---

검색 결과 게시글의 url과 제목, 본문, 갤리리 이름, 날짜를 크롤링 할 것이다. 최대한 많은 양을 수집하면 좋지만, 검색 결과에 노이즈가 발생하는 점을 고려하여, 정확도 순으로 10페이지를 수집하기로 하였다.

```python
from bs4 import BeautifulSoup
import requests as req
import pandas as pd
import urllib

search = input("검색어 입력: ")
uni_kor = urllib.parse.quote(search)

urls = []
titles = []
posts = []
gall_names = []
dates = []

header = {"User-Agent":"Mozilla/5.0"}

#링크 10페이지
for i in range(1, 11):
    url = "https://search.dcinside.com/post/p/" + str(i) + "/sort/accuracy/q/" + str(uni_kor)
    urls.append(url)
    for i in urls:
        header = {"User-Agent":"Mozilla/5.0"}
        res = req.get(i, headers=header)
        soup = BeautifulSoup(res.text, "html.parser")

        # 파싱 (제목, 글, 갤러리이름, 날짜)
        title = soup.find_all('a', attrs={'class': 'tit_txt'})
        for i in title:
            titles.append(i.get_text())
        post = soup.find_all('p', attrs={'class': 'link_dsc_txt'})
        for i in post:
            posts.append(i.get_text())

        gall_name = soup.select('a.sub_txt')
        for i in gall_name:
            gall_names.append(i.get_text())

        date = soup.select('span.date_time')
        for i in date:
            dates.append(i.get_text())

#본문 내용 노이즈 (갤러리이름, 날짜) 삭제 (홀수)
posts = posts[0::2]

# 데이터 프레임 만들기
df = pd.DataFrame({'date': dates, 'title': titles, 'content': posts, 'gall_name': gall_names})

df

df.to_csv(f'{search}.csv')
```

전체 코드가 길지 않아 전체 코드와 함께 설명을 하고자 한다. 일단 가장 먼저 url과 제목, 본문, 갤리리 이름, 날짜가 담길 빈 리스트를 생성하였다.

1페이지부터 10페이지 게시글을 전부 크롤링하기 위하여 for문을 활용하였다. range(1,11)은 1부터 11이 아닌 1부터 10을 의미한다. 또 유의할 점은 range(11)은 1부터 10이 아닌 0부터 10을 의미하기에 이에 유의해서 range()를 사용해야 한다.

1부터 10페이지의 링크에 접속하기 위해 다음과 같은 코드를 작성하였다.

```python
url = "https://search.dcinside.com/post/p/" + str(i) + "/sort/accuracy/q/" + str(uni_kor)
urls.append(url)
```
str(i)에 range(1,11)의 값이 반복해서 들어간다. 검색어로 입력했던 글이 유니코드로 변환되어 str(uni_kor) 부분에 들어가게 된다.
예를 들어, 서울공화국의 검색 결과 1페이지의 링크는 https://search.dcinside.com/post/p/1/sort/accuracy/q/.EC.84.9C.EC.9A.B8.EA.B3.B5.ED.99.94.EA.B5.AD ,

10페이지는 https://search.dcinside.com/post/p/10/sort/accuracy/q/.EC.84.9C.EC.9A.B8.EA.B3.B5.ED.99.94.EA.B5.AD 로 작성된다.

다음으로는 이렇게 작성된 링크를 BeautifulSoup로 크롤링한다.

```python
for i in urls:
    header = {"User-Agent":"Mozilla/5.0"}
    res = req.get(i, headers=header)
    soup = BeautifulSoup(res.text, "html.parser")
```

req.get('링크', headers=header)에 링크 부분에 생성한 urls가 들어가야 하므로 for문을 활용하였다. 따라서 링크 대신 i를 입력하였다.

```python
        # 파싱 (제목, 글, 갤러리이름, 날짜)
        title = soup.find_all('a', attrs={'class': 'tit_txt'})
        for i in title:
            titles.append(i.get_text())
        post = soup.find_all('p', attrs={'class': 'link_dsc_txt'})
        for i in post:
            posts.append(i.get_text())

        gall_name = soup.select('a.sub_txt')
        for i in gall_name:
            gall_names.append(i.get_text())

        date = soup.select('span.date_time')
        for i in date:
            dates.append(i.get_text())
```

위 코드를 통해 글의 제목, 본문, 갤러리 이름, 날짜를 크롤링하였다. 하지만 이렇게 태그를 가져오는 과정에서 하나의 문제가 발생했다. 본문을 가져올 때, 본문 내용 외에도 갤러리 이름과 날짜가 같이 수집되었다.

```python
#본문 내용 노이즈 (갤러리이름, 날짜) 삭제 (홀수)
posts = posts[0::2]
```

따라서 위와 같은 코드를 입력하여 본문 내용 안에 같이 수집된 갤러리 이름과 날짜를 삭제하였다. 수집하고자 했던 정보를 모두 크롤링 한 후에, 데이터 프레임을 생성하였고 csv파일로 저장하였다.

![Alt text](/images/t12.jpg)

디시인사이드 사이트에서 '서울공화국'을 검색한 결과의 csv파일이다. 총 1375개의 게시글이 수집된 것을 볼 수 있다. 이 코드를 활용하여 지역 혐오 단어 게시글을 수집한 후 지역 혐오 단어가 어떠한 맥락에서 주로 등장하는지 알아 볼 예정이다.