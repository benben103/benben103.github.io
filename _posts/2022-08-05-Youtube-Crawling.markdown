---
layout: post
title: Youtube Crawling (Subtitle)
image: 2.jpg
date: 2022-08-03 17:58:18 +0200
tags: [Project, Crawling]
categories: Project
---
유튜브 콘텐츠가 많아지면서 데이터 또한 다양해졌다. 유튜브 플랫폼은 영상 안 내용의 자막을 자동 생성하는 기능이 있다. 유튜브 영상 내 자막을 크롤링하기 위해 블로그를 참조하여 함수를 생성하였다.

#### Code
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
