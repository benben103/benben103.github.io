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
text = ''

for i in range(len(srt)):
            text += srt[i]['text'] + ' '
```

YouTubeTranscriptApi.get_transcript() 안에는 접속할 유튜브 영상 링크와 해당 언어를 위 코드와 같이 입력하면 된다. 

![Alt text](/images/p1.jpg)

유튜브 내 JTBC뉴스의 영상 하나를 예시로 가져왔다. 이 영상의 링크는 "https://www.youtube.com/watch?v=MPakXyjHybE"이다. 유튜브 영상의 링크는 "https://www.youtube.com/watch?v=특정값"으로 구성된다.
위 코드에 들어가는 링크는 특정값을 입력하면 된다. 예시 영상의 특정값은 "MPakXyjHybE"이다.

```python 
from youtube_transcript_api import YouTubeTranscriptApi

srt = YouTubeTranscriptApi.get_transcript("MPakXyjHybE", languages=['ko'])
text = ''

for i in range(len(srt)):
            text += srt[i]['text'] + ' '
```

자막 외에도 영상의 제목과 채널 이름, 영상 링크를 같이 크롤링하려고 한다. 이를 위해 BeautifulSoup 모듈을 사용한다. 크롤링에 대표적으로 쓰이는 모듈은 BeautifulSoup과 Selenium이 있다. 정적인 페이지에서는 BeautifulSoup을 주로 사용하고 동적인 페이지에서는 Selenium을 사용한다. 이와 관련한 자세한 내용은 추후에 게시글을 작성할 예정이다.

```python 
from bs4 import BeautifulSoup
import requests as req

h = {"User-Agent": "Mozilla/5.0"}
res = req.get("https://www.youtube.com/watch?v=MPakXyjHybE", headers = h)
soap = BeautifulSoup(res.text, "html.parser")
```

위의 코드가 BeautifulSoup을 사용할 때 주로 설정하는 내용이다.

{"User-Agent": "Mozilla/5.0"}은 크롤링 차단을 방지하는 코드이다. 서버는 일반 사용자와 로봇을 구분하여 차단할 수 있기 때문에, User Agent 정보를 만들어야 정상적으로 홈페이지에 접속이 가능하다.

requests는 입력한 홈페이지에 접속하여 정보를 가져오는 모듈이다. req.get("링크", headers=h)를 입력하면 되는데, 여기서 headers는 위에서 말한 User Agent 정보를 입력하는 것으로 이해하면 된다.

BeautifulSoup(res.text, "html.parser") 이 코드를 통해 해당 홈페이지의 html을 가져올 수 있다. html 외에도 XPath, JS Path 등을 가져올 수 있다. 

![Alt text](/images/p3.jpg)

크롤링을 위해서는 홈페이지의 html 구조에 대한 이해가 필요하다. BeautifulSoup 사용법 설명 글에 조금 더 자세한 설명을 담을 예정이다. 크롤링하고자 하는 정보를 찾는 방법은 매우 다양하기에, 이번 글에서는 해당 정보를 찾은 코드만 제공하고자 한다.

```python 
from bs4 import BeautifulSoup
import requests as req

h = {"User-Agent": "Mozilla/5.0"}
res = req.get("https://www.youtube.com/watch?v=MPakXyjHybE", headers = h)
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
```

제목과 링크, 채널 이름의 정보를 다음과 같은 코드로 추출하였다. 원하는 html 정보를 가져올 때 find(), select() 등 방법이 다양하기 때문에 위 코드보다 간단하게 작성할 수 있을 것이라 예상한다. 이에 대한 공부가 조금 더 필요하다 생각해 더 정보를 찾아보고자 한다. 


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
