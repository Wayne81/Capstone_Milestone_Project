# Data Science Capstone - Milestone Report
Wayne Chan  
2017年5月9日  



## Introduction


This document is a class assigment of Course named Data Science Capstone in the Data Science Specialization on Coursera, which is a great web based training course of data science.  

This project'purpose is to built a prediction algorithm about word text. It need to study the patterns from a corpus of documents and simulate people's writing behavior, then built up a prediction algorithm for another application that forsee user's most possible words when user enter a given word(s). 



## Loading and Reading Sourse data

The dataset was downloaded from the course repository available on: [Capstone Dataset](https://d396qusza40orc.cloudfront.net/dsscapstone/dataset/Coursera-SwiftKey.zip).

The data size is greater than 1GB. I have downloaded it and saved in the local PC for project programming.

There are 4 sets of files for 4 languages, but I only use English set for this project. It includes three files.
-en_US.blogs.txt
-en_US.news.txt
-en_US.twitter.txt.


```r
Blogs=read_lines("./final/en_US/en_US.blogs.txt") 
News=read_lines("./final/en_US/en_US.news.txt") 
Twitter=read_lines("./final/en_US/en_US.twitter.txt") 
```


## Basic Static of Source data


```r
# get the size information in MB
blog.size <- file.info("./final/en_US/en_US.blogs.txt")$size/1024^2
news.size <- file.info("./final/en_US/en_US.news.txt")$size/1024^2 
twit.size <- file.info("./final/en_US/en_US.twitter.txt")$size/1024^2

# Blogs 
blog.leng <- length(Blogs)
blog.sent <- sum(str_count(Blogs,boundary("sentence")))
blog.word <- sum(str_count(Blogs,boundary("word")))

# News
News.leng <- length(News)
News.sent <- sum(str_count(News,boundary("sentence")))
News.word <- sum(str_count(News,boundary("word")))

# twitter
twit.leng <- length(Twitter)
twit.sent <- sum(str_count(Twitter,boundary("sentence")))
twit.word <- sum(str_count(Twitter,boundary("word")))

# demonstrate static in table
knitr::kable(
data.frame(
  Data.File      = c("Blogs", "News", "Twitter"),
  File.Size.MB   = format(c(blog.size, news.size, twit.size), digits = 5),
  Total.Line     = format(c(blog.leng, News.leng, twit.leng), big.mark = ","),
  Total.Sentence = format(c(blog.sent, News.sent, twit.sent), big.mark = ","),
  Total.Word     = format(c(blog.word, News.word, twit.word), big.mark = ",")
))
```



Data.File   File.Size.MB   Total.Line   Total.Sentence   Total.Word 
----------  -------------  -----------  ---------------  -----------
Blogs       200.42         899,288      2,380,481        37,546,246 
News        196.28         1,010,242    2,025,776        34,762,395 
Twitter     159.36         2,360,148    3,780,372        30,093,369 

The static showst that 3 files totally around 600MB and around 4 million lines of data in total. Blogs is the biggest files in size, but twitter have most lines of records.



## Data Sampling

The data size will impact the performance of study, so that I will only use a subset of 15000 lines of sample for modeling.


```r
set.seed(2017)
Twit.Sample <- sample(Twitter, size=5000, replace = FALSE)
Blog.Sample <- sample(Blogs, size=5000, replace = FALSE)
News.Sample <- sample(News, size=5000, replace = FALSE)
Samples <- c(Twit.Sample, Blog.Sample, News.Sample)

writeLines(Samples, "./final/Samples.txt")
```


## Make Corpus and Data Cleansing  

This part will make the sampled data into a corpus and apply some data cleansing method to remove some non-interesting words and character. There are some process will apply in the corpus:

- Remove characters: /|@|\\|-...
- Convert to lowercase
- Remove bad words
- Common punctuation
- Removenumbers
- RemoveEnglish stop words
- strip whitespace
- stemming


```r
Samples <- iconv(Samples, 'UTF-8', 'ASCII', "byte")
Samples <- gsub(" #\\S*","", Samples)
Samples <- gsub("(f|ht)(tp)(s?)(://)(\\S*)", "", Samples)
Samples <- gsub("[^0-9A-Za-z///' ]", "", Samples)
Samples <- gsub("http[[:alnum:]]*", "", Samples)

BadWord = readLines('http://www.cs.cmu.edu/~biglou/resources/bad-words.txt')
Samples <- removeWords(Samples, BadWord)
Samples <- removeWords(Samples, stopwords("english"))
```



## N-gram Tokenization

Using quanteda package, we will create 3 N-grams models to explore word frequencies unigrams, bigrams and trigrams.


```r
# N-Gram = 1
dfmN1 <- dfm(Samples, ngrams=1, tolower=TRUE, remove_numbers=TRUE, 
             remove_punct=TRUE, remove_separators=TRUE, stem=TRUE, 
			 verbose=TRUE, remove=stopwords("english"))
textplot_wordcloud(dfmN1, random.order=TRUE, max.words=100, colors=brewer.pal(8,"Dark2"))
```

![](Milestone_Project_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

```r
# N-Gram = 2
dfmN2 <- dfm(Samples, ngrams=2, tolower=TRUE, remove_numbers=TRUE, 
             remove_punct=TRUE, remove_separators=TRUE, stem=TRUE, 
			 verbose=TRUE, remove=stopwords("english"))
textplot_wordcloud(dfmN2, random.order=TRUE, max.words=100, colors=brewer.pal(8,"Dark2"))
```

![](Milestone_Project_files/figure-html/unnamed-chunk-5-2.png)<!-- -->

```r
# N-Gram = 3
dfmN3 <- dfm(Samples, ngrams=3, tolower=TRUE, remove_numbers=TRUE, 
             remove_punct=TRUE, remove_separators=TRUE, stem=TRUE, 
			 verbose=TRUE, remove=stopwords("english"))
textplot_wordcloud(dfmN3, random.order=TRUE, max.words=100, colors=brewer.pal(8,"Dark2"))
```

![](Milestone_Project_files/figure-html/unnamed-chunk-5-3.png)<!-- -->


## Exploratory Data Analysis

Below wil explorate the 3 models of N-gram corpus and show the most frequent words in the study.


### Top 10 Word Frequencies of Unigrams, Bigrams and Trigrams


```r
# Ngram frequency counts for 1-word
N1Matrix <- as.data.frame(as.matrix(docfreq(dfmN1)))

# sorting the n-grams for plotting
N1Sort <- sort(rowSums(N1Matrix), decreasing=TRUE)
N1Freq <- data.frame(Words=names(N1Sort), Frequency=N1Sort)

g <- ggplot(head(N1Freq,10), aes(x=reorder(Words, Frequency), y=Frequency))
g <- g + geom_bar(stat="identity", fill="blue") + theme(axis.text.x=element_text(angle=45, hjust=1))
g <- g + ggtitle("Top 10 Tokens - nGrams = 1") + xlab("Unigrams") + ylab("Frequency")
g  
```

![](Milestone_Project_files/figure-html/unnamed-chunk-6-1.png)<!-- -->



```r
# Ngram frequency counts for 1-word
N2Matrix <- as.data.frame(as.matrix(docfreq(dfmN2)))

# sorting the n-grams for plotting
N2Sort <- sort(rowSums(N2Matrix), decreasing=TRUE)
N2Freq <- data.frame(Words=names(N2Sort), Frequency=N2Sort)

g <- ggplot(head(N2Freq,10), aes(x=reorder(Words, Frequency), y=Frequency))
g <- g + geom_bar(stat="identity", fill="blue") + theme(axis.text.x=element_text(angle=45, hjust=1))
g <- g + ggtitle("Top 10 Tokens - nGrams = 2") + xlab("Bigrams") + ylab("Frequency")
g  
```

![](Milestone_Project_files/figure-html/unnamed-chunk-7-1.png)<!-- -->


```r
# Ngram frequency counts for 1-word
N3Matrix <- as.data.frame(as.matrix(docfreq(dfmN3)))

# sorting the n-grams for plotting
N3Sort <- sort(rowSums(N3Matrix), decreasing=TRUE)
N3Freq <- data.frame(Words=names(N3Sort), Frequency=N3Sort)

g <- ggplot(head(N3Freq,10), aes(x=reorder(Words, Frequency), y=Frequency))
g <- g + geom_bar(stat="identity", fill="blue") + theme(axis.text.x=element_text(angle=45, hjust=1))
g <- g + ggtitle("Top 10 Tokens - nGrams = 3") + xlab("Trigrams") + ylab("Frequency")
g  
```

![](Milestone_Project_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

