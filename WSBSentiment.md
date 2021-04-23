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

Vader is an out-of-the-box sentiment analyzer packaged with NLTK. It takes sentence-tokenized paragraphs (or just individual sentences) and has a pre-trained sentiment for each word. Then it averages those sentiments to output the sentiment for the input sentence. This approach has some significant downsides, but seems to work reasonably well in practice (or at least that's what I read online).

```python
#Downloads and installs Vader sentiment analyzer
#import nltk
#nltk.download('vader_lexicon')

from nltk.sentiment.vader import SentimentIntensityAnalyzer
from nltk.tokenize import sent_tokenize

SA = SentimentIntensityAnalyzer()
```

```python
SA.polarity_scores('hello this is good thing')
```

## Adding Vader Sentiment to Existing Database Points

Since computing sentiment is a little expensive and storage of data is cheap, it makes the most sense to me to compute it once then store the sentiment of a piece of text alongside the text in the database.


### Comments:

```python
WSBCursor.execute('''SELECT comment_id, text FROM Comments''')

comments = []
for comment in WSBCursor:
    comments.append(comment)
```

```python
comment_frame = pd.DataFrame(comments,columns=['comment_id','text'])
sentiments = []
#loops over the data frame
for i in range(len(comment_frame)):
    ind_sentiments = []
    #Sentence tokenizes the comments then loops over the list of sentences
    for sentence in sent_tokenize(comment_frame['text'][i]):
        #Gets the sentiment for each sentence
        ind_sentiments.append(SA.polarity_scores(sentence)['compound'])
    
    #Appends the mean sentiment of each comment to the list
    sentiments.append(sum(ind_sentiments)/(len(ind_sentiments)))
#Creates a new row in the dataframe that includes the comment's sentiment
comment_frame['comment_sentiment'] = sentiments
```

```python
comment_frame
```

### Posts

Basically the same as comments, only with a column each for title and self text (if applicable)

```python
WSBCursor.execute('''SELECT post_id, post_title, self_text FROM Posts''')

posts = []
for post in WSBCursor:
    posts.append(post)
```

```python
post_frame = pd.DataFrame(posts,columns=['post_id','post_title','self_text'])

title_sentiments = []
self_sentiments = []
for i in range(len(post_frame)):
    #holds individual sentiments for one title or post
    ind_title = []
    ind_self = []
    for sentence in sent_tokenize(post_frame['post_title'][i]):
        ind_title.append(SA.polarity_scores(sentence)['compound'])
    
    #Appends mean sentiment of title
    title_sentiments.append(sum(ind_title)/len(ind_title))
    
    #Same thing for the self text
    for sentence in sent_tokenize(post_frame['self_text'][i]):
        ind_self.append(SA.polarity_scores(sentence)['compound'])
        
    #Appends mean sentiment of self text
    if (len(ind_self) == 0):
        self_sentiments.append(0)
    else:
        self_sentiments.append(sum(ind_self)/len(ind_self))
    
#Creates two new columns for the new sentiments
post_frame['post_title_sentiment'] = title_sentiments
post_frame['self_text_sentiment'] = self_sentiments
```

```python
post_frame
```

### Updating Existing Records

```python
#Updating comments first
comment_update_string = '''UPDATE Comments
                            SET comment_sentiment = {}
                            WHERE comment_id = {}'''
```

```python
for i in range(len(comment_frame)):
    WSBCursor.execute(comment_update_string.format(comment_frame['comment_sentiment'][i],comment_frame['comment_id'][i]))
    WSBDB.commit()
```

```python
#Then updating the posts
post_update_string = '''UPDATE Posts
                        SET post_title_sentiment = {}, self_text_sentiment = {}
                        WHERE post_id = {}'''
```

```python
post_frame.iloc[4950]
```

```python
#This randomly times out. I have no idea why, but if you just keep changing the loop indices and
#re-running it, you'll eventually cover the whole database
for i in range(4949,len(post_frame)):
    WSBCursor.execute(post_update_string.format(post_frame['post_title_sentiment'][i],post_frame['self_text_sentiment'][i],post_frame['post_id'][i]))
    WSBDB.commit()
```

## Summary

This ended up being a lot simpler than I expected, I just implemented this into the twice daily job that pulls data from the database.
