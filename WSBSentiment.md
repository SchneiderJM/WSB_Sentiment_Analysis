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
