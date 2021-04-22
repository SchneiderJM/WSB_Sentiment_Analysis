---
jupyter:
  jupytext:
    formats: ipynb,md
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.3'
      jupytext_version: 1.11.1
  kernelspec:
    display_name: Python 3
    language: python
    name: python3
---

# Sentiment Analysis of Stocks on WSB

Here, various sentiment analysis schemes are tested on various stocks found on WSB. 

```python
import mysql.connector
import csv
import pandas as pd
import pickle
#Loads in my db credentials
t = csv.reader(open('./dbcredentials.csv','r'),delimiter=',')
credentials = []
for credential in t:
    credentials.append(credential)
credentials = credentials[0]

WSBDB = mysql.connector.connect(
    host=credentials[0],
    user=credentials[1],
    password=credentials[2],
    database='WSB_Posts'
)

WSBCursor = WSBDB.cursor()
```

```python
#Gets a list of stopwords
words = stopwords.words('english')
words = list(words)
words.append('us')
```

# Vader

Vader is an out-of-the-box sentiment analyzer packaged with NLTK. It uses the sentence as a list of words and has a pre-trained sentiment for each word. Then it averages those sentiments to output the sentiment for your sentence. This approach has some significant downsides, but seems to work reasonably well in practice (or at least that's what I read online).

```python
#Downloads and installs Vader sentiment analyzer
#import nltk
#nltk.download('vader_lexicon')

from nltk.sentiment.vader import SentimentIntensityAnalyzer

SA = SentimentIntensityAnalyzer()
```

```python
SA.polarity_scores('not good')
```
