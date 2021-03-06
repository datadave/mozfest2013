# Sentence-level text analysis 

## Sentiment analysis in R

We're going to take advantage of a library that comes with a pretrained model, Tim Jurka's [sentiment](https://github.com/timjurka/sentiment) package. It's a perfectly suitable Naive Bayes classifier, but it is rather slow.  I don't recommend it for large datasets (read: millions of sentences).

Sentence-level annotations presumably translates well to other short forms of texts, such as microblogging (Twitter, Facebook statuses, etc).

We're going to use a very small random sample of the very famous `sentiment polarity dataset v1.0` by Pang and Lee, available [here](http://www.cs.cornell.edu/people/pabo/movie-review-data/).  This consists of thousands of processed sentences/snippets of movie reviews from the Rotten Tomatoes website.


```r
library(plyr)
library(reshape2)
library(sentiment)
```

```
## Loading required package: tm Loading required package: Rstem
```



# Sentiment

```r
# Using Tim Jurka's sentiment package Code adapted from Collingwood and
# Jurka
setwd("/Users/rweiss/Dropbox/presentations/MozFest2013/data/")
rt_neg = read.delim("rt-polaritydata/rt-polaritydata/rt-polarity.neg", header = F, 
    quote = "")
rt_pos = read.delim("rt-polaritydata/rt-polaritydata/rt-polarity.pos", header = F, 
    quote = "")
rt_neg = data.frame(rt_neg)
rt_pos = data.frame(rt_pos)
rt_neg$label = "negative"
rt_pos$label = "positive"
reviews = rbind(rt_neg, rt_pos)
names(reviews) = c("content", "label")

# let's start off with a small sample
sample_size = 100
num_documents = dim(reviews)[1]
reviews_sample <- reviews[sample(1:num_documents, size = sample_size, replace = FALSE), 
    ]
reviews_sample$content = as.character(reviews_sample$content)

foo = data.frame(as.character(reviews_sample$label), as.character(reviews_sample$content))
write.csv(reviews_sample$content, "../data/reviews_data.csv", row.names = F)
# let's save this so we can compare it against the SASA tool
write.table(foo, "../sentiment_examples/reviews_sample.csv", col.names = c("label", 
    "content"), row.names = F, quote = F, sep = "\t")

class(reviews_sample)  #make sure it is a data frame object
```

```
## [1] "data.frame"
```

```r
head(reviews_sample)  # Look at the first six lines or so
```

```
##                                                                                                                                                              content
## 5521  watching beanie and his gang put together his slasher video from spare parts and borrowed materials is as much fun as it must have been for them to make it . 
## 9550                                                                                       somehow ms . griffiths and mr . pryce bring off this wild welsh whimsy . 
## 10328                                                     [washington's] strong hand , keen eye , sweet spirit and good taste are reflected in almost every scene . 
## 9817                                                                 the salton sea has moments of inspired humour , though every scrap is of the darkest variety . 
## 6502                                            instead of a hyperbolic beat-charged urban western , it's an unpretentious , sociologically pointed slice of life . 
## 4940                                    a film that presents an interesting , even sexy premise then ruins itself with too many contrivances and goofy situations . 
##          label
## 5521  positive
## 9550  positive
## 10328 positive
## 9817  positive
## 6502  positive
## 4940  negative
```

```r
summary(reviews_sample)  #summarize the data
```

```
##    content             label          
##  Length:100         Length:100        
##  Class :character   Class :character  
##  Mode  :character   Mode  :character
```

```r
sapply(reviews_sample, class)  #look at the class of each column
```

```
##     content       label 
## "character" "character"
```

```r
dim(reviews_sample)  #Check the dimensions, rows and columns
```

```
## [1] 100   2
```

```r

predicted_sentiment = ddply(reviews_sample, .(content), function(x) {
    classify_polarity(x, algorithm = "bayes")
})

table(reviews_sample$label)
```

```
## 
## negative positive 
##       47       53
```

```r

predicted_sentiment$"POS/NEG" = as.numeric(as.character(predicted_sentiment$"POS/NEG"))
predicted_sentiment$label = cut(predicted_sentiment$"POS/NEG", breaks = c(0, 
    1, max(predicted_sentiment$"POS/NEG")))
levels(predicted_sentiment$label) = c("negative", "positive")
number_correct = sum(predicted_sentiment$label == reviews_sample$label)
number_correct/sample_size
```

```
## [1] 0.51
```

```r

table(reviews_sample$label)
```

```
## 
## negative positive 
##       47       53
```

```r
table(predicted_sentiment$label)
```

```
## 
## negative positive 
##       40       60
```

```r
reviews_labeled = data.frame(predicted_sentiment$label, predicted_sentiment$content)
write.csv(reviews_labeled, "../sentiment_examples/reviews_sample_2.csv", row.names = F)
```


## Big picture questions:
1. These models all have baseline accuracies measured against very famous, annotated datasets.  Therefore we have an estimate of how "accurate" a model should be. 
2. Go through the resulting predicted sentiment labels and examine whether you agree or disagree with them.  Count the proportion of values you agree with and then compare your agreement ratio agains the measured baseline accuracy.  How similar is it?
3. How appropriate is this model for this kind of data?  
4. What was this model trained on?  
5. How similar is the language of that training data against this movie review data?
6. How do these results compare against the other model's results?
7. We created a very simple bipolar classification.  By default, SASA will do positive, negative, neutral, and unsure.  Other models will do 5pt classification (very positive-very negative).  Still others will do discrete, categorical sentiment (see Wiebe's subjectivity lexicon).  There are many, many ways to label sentiment.  Which do you prefer?
