---
layout: post
title: 유튜브 자막 크롤링
image: 2.jpg
date: 2022-08-02 17:58:18 +0200
tags: [Project, Crawling]
categories: Project
---
유튜브 콘텐츠가 많아지면서 데이터 또한 다양해졌다. 유튜브는 자막을 자동 생성하는 기능을 제공하여 영상 안의 내용을 자막을 통해 추출할 수 있다. 유튜브 영상 내 자막을 크롤링하기 위해 여러 블로그를 참조하여 함수를 생성하였다. 

---
우선 유튜브 자막을 크롤링하기 위해서는 YoutubeTranscriptApi 모듈이 필요하다. 

![Alt text](/images/p2.jpg)
```python 
from youtube_transcript_api import YouTubeTranscriptApi

srt = YouTubeTranscriptApi.get_transcript("링크", languages=['ko'])
```

YouTubeTranscriptApi.get_transcript() 안에는 접속할 유튜브 영상 링크와 해당 언어를 위 코드와 같이 입력하면 된다. 

![Alt text](/images/p1.jpg)

유튜브 내 JTBC뉴스의 영상 하나를 예시로 가져왔다. 이 영상의 링크는 "https://www.youtube.com/watch?v=MPakXyjHybE"이다. 유튜브 영상의 링크는 "https://www.youtube.com/watch?v=특정값"으로 구성된다.
이를 이용하여 해당 링크에 접속하는 코드는 다음과 같다.

```python 
from youtube_transcript_api import YouTubeTranscriptApi
from bs4 import BeautifulSoup
import requests as req

def ytb_subtitle(start_url) : ### csv 파일 생성
    try:
        srt = YouTubeTranscriptApi.get_transcript(f"{start_url}", languages=['ko'])

        text = ''

        h = {"User-Agent": "Mozilla/5.0"}
        res = req.get(f"https://www.youtube.com/watch?v={start_url}", headers = h)
        soap = BeautifulSoup(res.text, "html.parser")

        title = soap.find('div').find('meta')
        title = str(title).replace('<meta content=','')
        title = title.replace(""" itemprop="name"/>""",'')

        link = soap.find('div').find('link')
        link = str(link).replace('link href=','')
        link = link.replace(""" itemprop="url"/>""",'')

        name = soap.find('div').find_all('link')
        name = str(name[2:3]).replace('[<link content=','')
        name = name.replace(""" itemprop="name"/>]""",'')

        for i in range(len(srt)):
            text += srt[i]['text'] + ' '

        df = pd.DataFrame({'title':[title], 'link':[link], 'name':[name], 'text': [text]})
        df.to_csv('YoutubeSubtitle.csv')

        print(title)
        print(link)
        print(name)
        print(df)
        print('O')
    except :
        print('X')

```

---
#### 전체 Code
```python 
from youtube_transcript_api import YouTubeTranscriptApi
from bs4 import BeautifulSoup
import requests as req
from konlpy.tag import Okt
import pandas as pd
import nltk
from nltk.tokenize import word_tokenize
from konlpy.tag import Mecab

"""유튜브 동영상 제목, 링크, 채널이름, 자막 크롤링"""

def ytb_subtitle(start_url) : ### csv 파일 생성
    try:
        srt = YouTubeTranscriptApi.get_transcript(f"{start_url}", languages=['ko'])

        text = ''

        h = {"User-Agent": "Mozilla/5.0"}
        res = req.get(f"https://www.youtube.com/watch?v={start_url}", headers = h)
        soap = BeautifulSoup(res.text, "html.parser")

        title = soap.find('div').find('meta')
        title = str(title).replace('<meta content=','')
        title = title.replace(""" itemprop="name"/>""",'')

        link = soap.find('div').find('link')
        link = str(link).replace('link href=','')
        link = link.replace(""" itemprop="url"/>""",'')

        name = soap.find('div').find_all('link')
        name = str(name[2:3]).replace('[<link content=','')
        name = name.replace(""" itemprop="name"/>]""",'')

        for i in range(len(srt)):
            text += srt[i]['text'] + ' '

        df = pd.DataFrame({'title':[title], 'link':[link], 'name':[name], 'text': [text]})
        df.to_csv('YoutubeSubtitle.csv')

        print(title)
        print(link)
        print(name)
        print(df)
        print('O')
    except :
        print('X')

def ytb_add(start_url) : ### 기존 csv 파일에 추가
    try:
        srt = YouTubeTranscriptApi.get_transcript(f"{start_url}", languages=['ko'])

        text = ''

        h = {"User-Agent": "Mozilla/5.0"}
        res = req.get(f"https://www.youtube.com/watch?v={start_url}", headers = h)
        soap = BeautifulSoup(res.text, "html.parser")

        title = soap.find('div').find('meta')
        title = str(title).replace('<meta content=','')
        title = title.replace(""" itemprop="name"/>""",'')

        link = soap.find('div').find('link')
        link = str(link).replace('link href=','')
        link = link.replace(""" itemprop="url"/>""",'')

        name = soap.find('div').find_all('link')
        name = str(name[2:3]).replace('[<link content=','')
        name = name.replace(""" itemprop="name"/>]""",'')

        for i in range(len(srt)):
            text += srt[i]['text'] + ' '

        old_df = pd.read_csv('YoutubeSubtitle.csv', encoding='utf-8')
        df = pd.DataFrame({'title':[title], 'link':[link], 'name':[name], 'text': [text]})
        new_df = old_df.append(df, ignore_index=True)
        new_df.to_csv('YoutubeSubtitle.csv')

        print(title)
        print(link)
        print(name)
        print(df)
        print('O')
    except :
        print('X')

```
