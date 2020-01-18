# Stop words {#stopwords}



Once we have tokenized text into words, it often becomes clear that not all of these words carry the same amount of information with them, if any information at all. Words that don't carry meaningful information are called **stop words**. It is common advice and practice to remove stop words for various NLP tasks, but the task of stop word removal is more nuanced than many resources may lead you to believe. In this chapter, we will investigate what a stop word list is, the differences between them, and the effects of using them in your preprocessing workflow.

The concept of stop words has a long history with Hans Peter Luhn credited with coining the term back in 1960 [@Luhn1960]. Examples of these words in English are "a", "the", "of", and "didn't"; these words are very common and don't seem to add much to the meaning of a text other than ensuring the structure of the sentence is sound. 

<div class="rmdtip">
<p>Thinking of words as being either informative or non-informative is quite limiting, and we prefer to consider words as having a more fluid or continuous amount of information associated with them, where this information is context-specific as well.</p>
</div>

Historically, one of the main reasons for removing stop words was to decrease computational time for text mining; it can be regarded as a dimensionality reduction of text data and was commonly used in search engines to give better results [@Huston2010].

## Using premade stop word lists

A quick solution to getting a list of stop words is to use one that is already created for you. This is appealing because it requires a low level of effort, but beware that not all lists are created equal. @nothman-etal-2018-stop found some alarming results in a study of 52 stop word lists available in open-source software packages. Their unexpected findings included how different stop word lists have a varying number of words depending on the specificity of the list. Among some of the more grave issues were misspellings ("fify" instead of "fifty"), the inclusion of clearly informative words such as "computer" and "cry", and various internal inconsistencies such as including the word "has" but not the word "does". This is not to say that you should never use a stop word list that has been included in an open-source software project. However, you should always inspect and verify the list you are using, both to make sure it hasn't changed since you used it last, and also to check that it is appropriate for your use case.

There is a broad selection of stop word lists available today. For the purpose of this chapter we will focus on three lists of English stop words provided by the **stopwords** package [@R-stopwords]. The first is from the SMART (System for the Mechanical Analysis and Retrieval of Text) Information Retrieval System, an information retrieval system developed at Cornell University in the 1960s [@Lewis2014]. The second is the English Snowball stop word list [@porter2001snowball], and the last is the English list from the [Stopwords ISO](https://github.com/stopwords-iso/stopwords-iso) collection. These stop word lists are all considered general purpose and not domain specific.

Before we start delving into the content inside the lists, let's take a look at how many words are included in each.


```r
library(stopwords)
length(stopwords(source = "smart"))
length(stopwords(source = "snowball"))
length(stopwords(source = "stopwords-iso"))
```

```
## [1] 571
## [1] 175
## [1] 1298
```

The length of these lists are quite varied, with the longest list being over seven times longer than the shortest! Let's examine the overlap of the words that appear in the three lists in Figure \@ref(fig:stopwordoverlap).

<div class="figure">
<img src="stopwords_files/figure-epub3/stopwordoverlap-1.png" alt="Set intersections for three common stop word lists"  />
<p class="caption">(\#fig:stopwordoverlap)Set intersections for three common stop word lists</p>
</div>

These three lists are almost true subsets of each other. The only excepetion is a set of ten words that appear in Snowball and ISO but not in the SMART list. What are those words?


```r
setdiff(
  stopwords(source = "snowball"),
  stopwords(source = "smart")
)
```

```
##  [1] "she's"   "he'd"    "she'd"   "he'll"   "she'll"  "shan't"  "mustn't"
##  [8] "when's"  "why's"   "how's"
```

All these words are contractions. This is *not* because the SMART lexicon doesn't include contractions, because if we look there are almost fifty of them.


```r
str_subset(stopwords(source = "smart"), "'")
```

```
##  [1] "a's"       "ain't"     "aren't"    "c'mon"     "c's"       "can't"    
##  [7] "couldn't"  "didn't"    "doesn't"   "don't"     "hadn't"    "hasn't"   
## [13] "haven't"   "he's"      "here's"    "i'd"       "i'll"      "i'm"      
## [19] "i've"      "isn't"     "it'd"      "it'll"     "it's"      "let's"    
## [25] "shouldn't" "t's"       "that's"    "there's"   "they'd"    "they'll"  
## [31] "they're"   "they've"   "wasn't"    "we'd"      "we'll"     "we're"    
## [37] "we've"     "weren't"   "what's"    "where's"   "who's"     "won't"    
## [43] "wouldn't"  "you'd"     "you'll"    "you're"    "you've"
```

We seem to have stumbled upon an inconsistency; why does SMART include `"he's"` but not `"she's"`? It is hard to say, but this would be worth rectifying before applying these stop word lists to an analysis or model preprocessing. It is likely that this stop word list was generated by selecting the most frequent words across a large corpus of text that had more representation for text about men than women. This is once again a reminder that we should always look carefully at the premade word lists and other artifacts we use to make sure it works well with our needs. 

<div class="rmdtip">
<p>It is perfectly acceptable to start with a premade word list and remove or append additional words according to your particular use case.</p>
</div>


When you select a stop word list, it is important that you consider its size and breadth. Having a small and concise list of words can moderately reduce your token count while not having too great of an influence on your models, assuming that you picked appropriate words. As the size of your stop word list grows, each word added will have a diminishing positive effect with the increasing risk that a meaningful word has been placed on the list by mistake. In a later chapter on model building, we will show an example where we analyze the effects of different stop word lists.

### Stop word removal in R

Now that we have some stop word lists, we can move forward with removing these words. The particular way we remove stop words depends on the shape of our data. If you have your text in a tidy format with one word per row, you can use `filter()` from **dplyr** with a negated `%in%` if you have the stop words as a vector, or you can use `anti_join()` from **dplyr** if the stop words are in a `tibble()`. Like in our previous chapter, let's examine the text of "The Fir-Tree" by Hans Christian Andersen, and use **tidytext** to tokenize the text into words.


```r
library(hcandersenr)
library(tidyverse)
library(tidytext)

fir_tree <- hca_fairytales() %>%
  filter(
    book == "The fir tree",
    language == "English"
  )

tidy_fir_tree <- fir_tree %>%
  unnest_tokens(word, text)
```

And we can use the Snowball stop word list as an example. Since the stop words return from this function as a vector, we will use `filter()`.


```r
tidy_fir_tree %>%
  filter(!(tidy_fir_tree$word %in% stopwords(source = "snowball")))
```

```
## # A tibble: 1,547 x 3
##    book         language word   
##    <chr>        <chr>    <chr>  
##  1 The fir tree English  far    
##  2 The fir tree English  forest 
##  3 The fir tree English  warm   
##  4 The fir tree English  sun    
##  5 The fir tree English  fresh  
##  6 The fir tree English  air    
##  7 The fir tree English  made   
##  8 The fir tree English  sweet  
##  9 The fir tree English  resting
## 10 The fir tree English  place  
## # … with 1,537 more rows
```

If we use the `get_stopwords()` function from **tidytext** instead, then we can use the `anti_join()` function.


```r
tidy_fir_tree %>%
  anti_join(get_stopwords(source = "snowball"))
```

```
## # A tibble: 1,547 x 3
##    book         language word   
##    <chr>        <chr>    <chr>  
##  1 The fir tree English  far    
##  2 The fir tree English  forest 
##  3 The fir tree English  warm   
##  4 The fir tree English  sun    
##  5 The fir tree English  fresh  
##  6 The fir tree English  air    
##  7 The fir tree English  made   
##  8 The fir tree English  sweet  
##  9 The fir tree English  resting
## 10 The fir tree English  place  
## # … with 1,537 more rows
```

The result of these two stop word removals is the same since we used the same stop word list in both cases.

## Creating your own stop words list

Another way to get a stop word list is to create one yourself. Let's explore a few different ways to find appropriate words to use. We will use the tokenized data from "The Fir-Tree" as a first example. Let's take the words and rank them by their count or frequency.

<div class="figure">
<img src="stopwords_files/figure-epub3/unnamed-chunk-9-1.png" alt="We counted words in &quot;The Fir Tree&quot; and ordered them by count or frequency."  />
<p class="caption">(\#fig:unnamed-chunk-9)We counted words in "The Fir Tree" and ordered them by count or frequency.</p>
</div>

We recognize many of what we would consider stop words in the first column here, with three big exceptions. We see `"tree"` at 3, `"fir"` at 12 and `"little"` at 22. These words appear high on our list but do provide valuable information as they all reference the main character. What went wrong with this approach? Creating a stop word list using high-frequency words works best when it is created on a **corpus** of documents, not individual documents. This is because the words found in a single document will be document specific and the overall pattern of words will not generalize that well. 

<div class="rmdnote">
<p>In NLP, a corpus is a set of texts or documents. The set of Hans Christian Andersen's fairy tales can be considered a corpus, with each fairy tale a document within that corpus. The set of United States Supreme Court opinions can be considered a different corpus, with each written opinion being a document within <em>that</em> corpus.</p>
</div>

The word `"tree"` does seem important as it is about the main character, but it could also be appearing so often that it stops providing any information. Let's try a different approach, extracting high-frequency words from the corpus of *all* English fairy tales by H.C. Andersen.

<div class="figure">
<img src="stopwords_files/figure-epub3/unnamed-chunk-11-1.png" alt="We counted words in all English fairy tales by Hans Christian Andersen and ordered them by count or frequency."  />
<p class="caption">(\#fig:unnamed-chunk-11)We counted words in all English fairy tales by Hans Christian Andersen and ordered them by count or frequency.</p>
</div>

This list is more appropriate for our concept of stop words, and now it is time for us to make some choices. How many do we want to include in our stop word list? Which words should we add and/or remove based on prior information? Selecting the number of words to remove is best done by a case-by-case basis as it can be difficult to determine apriori how many different "meaningless" words appear in a corpus. Our suggestion is to start with a low number like twenty and increase by ten words until you get to words that are not appropriate as stop words for your analytical purpose. 

It is worth keeping in mind that this list is not perfect. It is based on the corpus of documents we had available, which is potentially biased since all the fairy tales were written by the same European white male from the early 1800s. 

<div class="rmdtip">
<p>This bias can be minimized by removing words we would expect to be over-represented or to add words we expect to be under-represented.</p>
</div>

Easy examples are to include the compliments to the words in the lists if they are not present. Include `"big"` if `"small"` is present, `"old"` if `"young"` is present. This example list has words associated with women often listed lower in rank than words associated with men. With `"man"` being at rank 79, but `"woman"` at rank 179, choosing a threshold of 100 would lead to only one of these words be included. Depending on how important you think such nouns are going to be in your texts, either add `"woman"` or delete `"man"`.

Figure \@ref(fig:genderrank) shows how the words associated with men have higher rank than the words associated with women. By using a single threshold to create a stop word list, you would likely only include one form of such words.

<div class="figure">
<img src="stopwords_files/figure-epub3/genderrank-1.png" alt="We counted tokens and ranked according to total. Rank 1 has most occurrences."  />
<p class="caption">(\#fig:genderrank)We counted tokens and ranked according to total. Rank 1 has most occurrences.</p>
</div>

Imagine now we would like to create a stop word list that spans multiple different genres, in such a way that the subject-specific stop words don't overlap. For this case, we would like words to be denoted as a stop word only if it is a stop word in all the genres. You could find the words individually in each genre and using the right intersections. However, that approach might take a substantial amount of time.

Below is a bad example where we try to create a multi-language list of stop words. To accomplish this we calculate the [inverse document frequency](https://www.tidytextmining.com/tfidf.html) (IDF) of each word, and create the stop word list based on the words with the lowest IDF. The following function takes a tokenized dataframe and returns a dataframe with a column for each word and a column for the IDF.


```r
library(rlang)
calc_idf <- function(df, word, document) {
  words <- df %>%
    pull({{ word }}) %>%
    unique()

  n_docs <- length(unique(pull(df, {{ document }})))

  n_words <- df %>%
    nest(data = c({{ word }})) %>%
    pull(data) %>%
    map_dfc(~ words %in% unique(pull(.x, {{ word }}))) %>%
    rowSums()

  tibble(
    word = words,
    idf = log(n_docs / n_words)
  )
}
```

Here is the result where we try to create a cross-language list of stop words, by taking each fairy tale as a document. It is not very good! The overlap between what words appear in each language is very small, and that is what we mostly see in this list.

<div class="figure">
<img src="stopwords_files/figure-epub3/unnamed-chunk-14-1.png" alt="We counted words from all of H.C. Andersen's fairy tales in Danish, English, French, German, and Spanish and ordered by count or frequency."  />
<p class="caption">(\#fig:unnamed-chunk-14)We counted words from all of H.C. Andersen's fairy tales in Danish, English, French, German, and Spanish and ordered by count or frequency.</p>
</div>

TODO do same example with English only.

do MP, VP and  SAT 
https://pdfs.semanticscholar.org/c543/8e216071f6180c228cc557fb1d3c77edb3a3.pdf

## All stop word lists are context specific

Since all work related to the text is specific to context, it is important to make sure that the stop word list you use reflects the word space that you are planning on using it on. One common concern to consider is how pronouns bring information to your text. Pronouns are included in many different stop word lists (although inconsistently) and they will often *not* be noise in text data.

On the other hand, sometimes you will have to add in words yourself, depending on the domain. If you are working with texts for dessert recipes, certain ingredients (sugar, eggs, water) and actions (whisking, baking, stirring) may be frequent enough to pass your stop word threshold, but it's possible you will want to keep them as they may be informative. Throwing away "eggs" as a common word would make it harder or downright impossible to determine if certain recipes are vegan or not, while whisking and stirring may be fine to remove as distinguishing between recipes that do and don't require a whisk might not be that big of a deal.

## What happens when you remove stop words

We have discussed different ways of finding and removing stop words; now let's see what happens once you do remove them. First, let's explore the impact of the number of words that are included in the list. Figure \@ref(fig:stopwordresults) shows what percentage of words are removed as a function of the number of words in a text. The different colors represent the 3 different stop word lists we have considered in this chapter.

<div class="figure">
<img src="stopwords_files/figure-epub3/stopwordresults-1.png" alt="Proportion of words removed for different stop word lists and different document lengths"  />
<p class="caption">(\#fig:stopwordresults)Proportion of words removed for different stop word lists and different document lengths</p>
</div>

We notice, as we would predict, that larger stop word lists remove more words then shorter stop word lists. In this example with fairy tales, over half of the words have been removed, with the largest list removing over 80% of the words. We observe that shorter texts have a lower percentage of stop words. Since we are looking at fairy tales, this could be explained by the fact that a story has to be told regardless of the length of the fairy tale, so shorter texts are going to be more dense with more informative words.

Another problem you might have is dealing with misspellings. 

<div class="rmdwarning">
<p>Most premade stop word lists assume that all the words are spelled correctly.</p>
</div>

Handling misspellings when using premade lists can be done by manually adding common misspellings. You could imagine creating all words that are a certain string distance away from the stop words, but we do not recommend this as you would quickly include informative words this way.

One of the downsides of creating your own stop word lists using frequencies is that you are limited to using words that you have already observed. It could happen that `"she'd"` is included in your training corpus but the word `"he'd"` did not reach the threshold This is a case where you need to look at your words and adjust accordingly. Here the large premade stop word lists can serve as inspiration for missing words.

In a later chapter (TODO add link) will we investigate the influence of removing stop words in the context of modeling. Given the right list of words, you see no harm to the model performance, and may even see improvement in result due to noise reduction [@Feldman2007].

## Stop words in languages other than English

So far in this chapter, we have been spent the majority of the time on the English language, but English is not representative of every language. The stop word lists we examined in this chapter have been English and the notion of "short" and "long" lists we have used here are specific to English as a language. You should expect different languages to have a varying number of "uninformative" words, and for this number to depend on the morphological richness of a language; lists that contain all possible morphological variants of each stop word could become quite large.

Different languages have different numbers of words in each class of words. An example is how the grammatical case influences the articles used in German. Below are a couple of diagrams showing the use of definite and indefinite articles in German. Notice how German nouns have three genders (masculine, feminine, and neuter), which are not uncommon in languages around the world. Articles are almost always considered as stop words in English as they carry very little information. However, German articles give some indication of the case which can be used when selecting a list of stop words in German or any other language where the grammatical case is reflected in the text.

<!--html_preserve--><style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#zpvnpvqdtb .gt_table {
  display: table;
  border-collapse: collapse;
  margin-left: auto;
  /* table.margin.left */
  margin-right: auto;
  /* table.margin.right */
  color: #333333;
  font-size: 16px;
  /* table.font.size */
  background-color: #FFFFFF;
  /* table.background.color */
  width: auto;
  /* table.width */
  border-top-style: solid;
  /* table.border.top.style */
  border-top-width: 2px;
  /* table.border.top.width */
  border-top-color: #A8A8A8;
  /* table.border.top.color */
  border-bottom-style: solid;
  /* table.border.bottom.style */
  border-bottom-width: 2px;
  /* table.border.bottom.width */
  border-bottom-color: #A8A8A8;
  /* table.border.bottom.color */
}

#zpvnpvqdtb .gt_heading {
  background-color: #FFFFFF;
  /* heading.background.color */
  border-bottom-color: #FFFFFF;
  /* table.background.color */
  border-left-style: hidden;
  /* heading.border.lr.style */
  border-left-width: 1px;
  /* heading.border.lr.width */
  border-left-color: #D3D3D3;
  /* heading.border.lr.color */
  border-right-style: hidden;
  /* heading.border.lr.style */
  border-right-width: 1px;
  /* heading.border.lr.width */
  border-right-color: #D3D3D3;
  /* heading.border.lr.color */
}

#zpvnpvqdtb .gt_title {
  color: #333333;
  font-size: 125%;
  /* heading.title.font.size */
  font-weight: initial;
  /* heading.title.font.weight */
  padding-top: 4px;
  /* heading.top.padding - not yet used */
  padding-bottom: 4px;
  border-bottom-color: #FFFFFF;
  /* table.background.color */
  border-bottom-width: 0;
}

#zpvnpvqdtb .gt_subtitle {
  color: #333333;
  font-size: 85%;
  /* heading.subtitle.font.size */
  font-weight: initial;
  /* heading.subtitle.font.weight */
  padding-top: 0;
  padding-bottom: 4px;
  /* heading.bottom.padding - not yet used */
  border-top-color: #FFFFFF;
  /* table.background.color */
  border-top-width: 0;
}

#zpvnpvqdtb .gt_bottom_border {
  border-bottom-style: solid;
  /* heading.border.bottom.style */
  border-bottom-width: 2px;
  /* heading.border.bottom.width */
  border-bottom-color: #D3D3D3;
  /* heading.border.bottom.color */
}

#zpvnpvqdtb .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  padding-top: 4px;
  padding-bottom: 4px;
}

#zpvnpvqdtb .gt_col_headings {
  border-top-style: solid;
  /* column_labels.border.top.style */
  border-top-width: 2px;
  /* column_labels.border.top.width */
  border-top-color: #D3D3D3;
  /* column_labels.border.top.color */
  border-bottom-style: solid;
  /* column_labels.border.bottom.style */
  border-bottom-width: 2px;
  /* column_labels.border.bottom.width */
  border-bottom-color: #D3D3D3;
  /* column_labels.border.bottom.color */
  border-left-style: none;
  /* column_labels.border.lr.style */
  border-left-width: 1px;
  /* column_labels.border.lr.width */
  border-left-color: #D3D3D3;
  /* column_labels.border.lr.color */
  border-right-style: none;
  /* column_labels.border.lr.style */
  border-right-width: 1px;
  /* column_labels.border.lr.width */
  border-right-color: #D3D3D3;
  /* column_labels.border.lr.color */
}

#zpvnpvqdtb .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  /* column_labels.background.color */
  font-size: 100%;
  /* column_labels.font.size */
  font-weight: normal;
  /* column_labels.font.weight */
  text-transform: inherit;
  /* column_labels.text_transform */
  vertical-align: middle;
  padding: 5px;
  margin: 10px;
  overflow-x: hidden;
}

#zpvnpvqdtb .gt_sep_right {
  border-right: 5px solid #FFFFFF;
}

#zpvnpvqdtb .gt_group_heading {
  padding: 8px;
  /* row_group.padding */
  color: #333333;
  background-color: #FFFFFF;
  /* row_group.background.color */
  font-size: 100%;
  /* row_group.font.size */
  font-weight: initial;
  /* row_group.font.weight */
  text-transform: inherit;
  /* row_group.text_transform */
  border-top-style: solid;
  /* row_group.border.top.style */
  border-top-width: 2px;
  /* row_group.border.top.width */
  border-top-color: #D3D3D3;
  /* row_group.border.top.color */
  border-bottom-style: solid;
  /* row_group.border.bottom.style */
  border-bottom-width: 2px;
  /* row_group.border.bottom.width */
  border-bottom-color: #D3D3D3;
  /* row_group.border.bottom.color */
  border-left-style: none;
  /* row_group.border.left.style */
  border-left-width: 1px;
  /* row_group.border.left.width */
  border-left-color: #D3D3D3;
  /* row_group.border.left.color */
  border-right-style: none;
  /* row_group.border.right.style */
  border-right-width: 1px;
  /* row_group.border.right.width */
  border-right-color: #D3D3D3;
  /* row_group.border.right.color */
  vertical-align: middle;
}

#zpvnpvqdtb .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  /* row_group.background.color */
  font-size: 100%;
  /* row_group.font.size */
  font-weight: initial;
  /* row_group.font.weight */
  border-top-style: solid;
  /* row_group.border.top.style */
  border-top-width: 2px;
  /* row_group.border.top.width */
  border-top-color: #D3D3D3;
  /* row_group.border.top.color */
  border-bottom-style: solid;
  /* row_group.border.bottom.style */
  border-bottom-width: 2px;
  /* row_group.border.bottom.width */
  border-bottom-color: #D3D3D3;
  /* row_group.border.bottom.color */
  vertical-align: middle;
}

#zpvnpvqdtb .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
  /* row.striping.background_color */
}

#zpvnpvqdtb .gt_from_md > :first-child {
  margin-top: 0;
}

#zpvnpvqdtb .gt_from_md > :last-child {
  margin-bottom: 0;
}

#zpvnpvqdtb .gt_row {
  padding-top: 8px;
  /* data_row.padding */
  padding-bottom: 8px;
  /* data_row.padding */
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  /* table_body.hlines.style */
  border-top-width: 1px;
  /* table_body.hlines.width */
  border-top-color: #D3D3D3;
  /* table_body.hlines.color */
  border-left-style: none;
  /* table_body.vlines.style */
  border-left-width: 1px;
  /* table_body.vlines.width */
  border-left-color: #D3D3D3;
  /* table_body.vlines.color */
  border-right-style: none;
  /* table_body.vlines.style */
  border-right-width: 1px;
  /* table_body.vlines.width */
  border-right-color: #D3D3D3;
  /* table_body.vlines.color */
  vertical-align: middle;
  overflow-x: hidden;
}

#zpvnpvqdtb .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  /* stub.background.color */
  font-weight: initial;
  /* stub.font.weight */
  text-transform: inherit;
  /* stub.text_transform */
  border-right-style: solid;
  /* stub.border.style */
  border-right-width: 2px;
  /* stub.border.width */
  border-right-color: #D3D3D3;
  /* stub.border.color */
  padding-left: 12px;
}

#zpvnpvqdtb .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  /* summary_row.background.color */
  text-transform: inherit;
  /* summary_row.text_transform */
  padding-top: 8px;
  /* summary_row.padding */
  padding-bottom: 8px;
  /* summary_row.padding */
  padding-left: 5px;
  padding-right: 5px;
}

#zpvnpvqdtb .gt_first_summary_row {
  padding-top: 8px;
  /* summary_row.padding */
  padding-bottom: 8px;
  /* summary_row.padding */
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: solid;
  /* summary_row.border.style */
  border-top-width: 2px;
  /* summary_row.border.width */
  border-top-color: #D3D3D3;
  /* summary_row.border.color */
}

#zpvnpvqdtb .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  /* grand_summary_row.background.color */
  text-transform: inherit;
  /* grand_summary_row.text_transform */
  padding-top: 8px;
  /* grand_summary_row.padding */
  padding-bottom: 8px;
  /* grand_summary_row.padding */
  padding-left: 5px;
  padding-right: 5px;
}

#zpvnpvqdtb .gt_first_grand_summary_row {
  padding-top: 8px;
  /* grand_summary_row.padding */
  padding-bottom: 8px;
  /* grand_summary_row.padding */
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  /* grand_summary_row.border.style */
  border-top-width: 6px;
  /* grand_summary_row.border.width */
  border-top-color: #D3D3D3;
  /* grand_summary_row.border.color */
}

#zpvnpvqdtb .gt_table_body {
  border-top-style: solid;
  /* table_body.border.top.style */
  border-top-width: 2px;
  /* table_body.border.top.width */
  border-top-color: #D3D3D3;
  /* table_body.border.top.color */
  border-bottom-style: solid;
  /* table_body.border.bottom.style */
  border-bottom-width: 2px;
  /* table_body.border.bottom.width */
  border-bottom-color: #D3D3D3;
  /* table_body.border.bottom.color */
}

#zpvnpvqdtb .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  /* footnotes.background.color */
  border-bottom-style: none;
  /* footnotes.border.bottom.style */
  border-bottom-width: 2px;
  /* footnotes.border.bottom.width */
  border-bottom-color: #D3D3D3;
  /* footnotes.border.bottom.color */
  border-left-style: none;
  /* footnotes.border.lr.color */
  border-left-width: 2px;
  /* footnotes.border.lr.color */
  border-left-color: #D3D3D3;
  /* footnotes.border.lr.color */
  border-right-style: none;
  /* footnotes.border.lr.color */
  border-right-width: 2px;
  /* footnotes.border.lr.color */
  border-right-color: #D3D3D3;
  /* footnotes.border.lr.color */
}

#zpvnpvqdtb .gt_footnote {
  margin: 0px;
  font-size: 90%;
  /* footnotes.font.size */
  padding: 4px;
  /* footnotes.padding */
}

#zpvnpvqdtb .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  /* source_notes.background.color */
  border-bottom-style: none;
  /* source_notes.border.bottom.style */
  border-bottom-width: 2px;
  /* source_notes.border.bottom.width */
  border-bottom-color: #D3D3D3;
  /* source_notes.border.bottom.color */
  border-left-style: none;
  /* source_notes.border.lr.style */
  border-left-width: 2px;
  /* source_notes.border.lr.style */
  border-left-color: #D3D3D3;
  /* source_notes.border.lr.style */
  border-right-style: none;
  /* source_notes.border.lr.style */
  border-right-width: 2px;
  /* source_notes.border.lr.style */
  border-right-color: #D3D3D3;
  /* source_notes.border.lr.style */
}

#zpvnpvqdtb .gt_sourcenote {
  font-size: 90%;
  /* source_notes.font.size */
  padding: 4px;
  /* source_notes.padding */
}

#zpvnpvqdtb .gt_left {
  text-align: left;
}

#zpvnpvqdtb .gt_center {
  text-align: center;
}

#zpvnpvqdtb .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#zpvnpvqdtb .gt_font_normal {
  font-weight: normal;
}

#zpvnpvqdtb .gt_font_bold {
  font-weight: bold;
}

#zpvnpvqdtb .gt_font_italic {
  font-style: italic;
}

#zpvnpvqdtb .gt_super {
  font-size: 65%;
}

#zpvnpvqdtb .gt_footnote_marks {
  font-style: italic;
  font-size: 65%;
}
</style>
<div id="zpvnpvqdtb" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;"><table class="gt_table">
  <thead class="gt_header">
    <tr>
      <th colspan="5" class="gt_heading gt_title gt_font_normal gt_center" style>German Definite Articles (the)</th>
    </tr>
    <tr>
      <th colspan="5" class="gt_heading gt_subtitle gt_font_normal gt_center gt_bottom_border" style></th>
    </tr>
  </thead>
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1"></th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1">Masculine</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1">Feminine</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1">Neuter</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1">Plural</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr>
      <td class="gt_row gt_left gt_stub">Nominative</td>
      <td class="gt_row gt_left">der</td>
      <td class="gt_row gt_left">die</td>
      <td class="gt_row gt_left">das</td>
      <td class="gt_row gt_left">die</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Accusative</td>
      <td class="gt_row gt_left gt_striped">den</td>
      <td class="gt_row gt_left gt_striped">die</td>
      <td class="gt_row gt_left gt_striped">das</td>
      <td class="gt_row gt_left gt_striped">die</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Dative</td>
      <td class="gt_row gt_left">dem</td>
      <td class="gt_row gt_left">der</td>
      <td class="gt_row gt_left">dem</td>
      <td class="gt_row gt_left">den</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Genitive</td>
      <td class="gt_row gt_left gt_striped">des</td>
      <td class="gt_row gt_left gt_striped">der</td>
      <td class="gt_row gt_left gt_striped">des</td>
      <td class="gt_row gt_left gt_striped">der</td>
    </tr>
  </tbody>
  
  
</table></div><!--/html_preserve-->

<!--html_preserve--><style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#auhridjtrm .gt_table {
  display: table;
  border-collapse: collapse;
  margin-left: auto;
  /* table.margin.left */
  margin-right: auto;
  /* table.margin.right */
  color: #333333;
  font-size: 16px;
  /* table.font.size */
  background-color: #FFFFFF;
  /* table.background.color */
  width: auto;
  /* table.width */
  border-top-style: solid;
  /* table.border.top.style */
  border-top-width: 2px;
  /* table.border.top.width */
  border-top-color: #A8A8A8;
  /* table.border.top.color */
  border-bottom-style: solid;
  /* table.border.bottom.style */
  border-bottom-width: 2px;
  /* table.border.bottom.width */
  border-bottom-color: #A8A8A8;
  /* table.border.bottom.color */
}

#auhridjtrm .gt_heading {
  background-color: #FFFFFF;
  /* heading.background.color */
  border-bottom-color: #FFFFFF;
  /* table.background.color */
  border-left-style: hidden;
  /* heading.border.lr.style */
  border-left-width: 1px;
  /* heading.border.lr.width */
  border-left-color: #D3D3D3;
  /* heading.border.lr.color */
  border-right-style: hidden;
  /* heading.border.lr.style */
  border-right-width: 1px;
  /* heading.border.lr.width */
  border-right-color: #D3D3D3;
  /* heading.border.lr.color */
}

#auhridjtrm .gt_title {
  color: #333333;
  font-size: 125%;
  /* heading.title.font.size */
  font-weight: initial;
  /* heading.title.font.weight */
  padding-top: 4px;
  /* heading.top.padding - not yet used */
  padding-bottom: 4px;
  border-bottom-color: #FFFFFF;
  /* table.background.color */
  border-bottom-width: 0;
}

#auhridjtrm .gt_subtitle {
  color: #333333;
  font-size: 85%;
  /* heading.subtitle.font.size */
  font-weight: initial;
  /* heading.subtitle.font.weight */
  padding-top: 0;
  padding-bottom: 4px;
  /* heading.bottom.padding - not yet used */
  border-top-color: #FFFFFF;
  /* table.background.color */
  border-top-width: 0;
}

#auhridjtrm .gt_bottom_border {
  border-bottom-style: solid;
  /* heading.border.bottom.style */
  border-bottom-width: 2px;
  /* heading.border.bottom.width */
  border-bottom-color: #D3D3D3;
  /* heading.border.bottom.color */
}

#auhridjtrm .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  padding-top: 4px;
  padding-bottom: 4px;
}

#auhridjtrm .gt_col_headings {
  border-top-style: solid;
  /* column_labels.border.top.style */
  border-top-width: 2px;
  /* column_labels.border.top.width */
  border-top-color: #D3D3D3;
  /* column_labels.border.top.color */
  border-bottom-style: solid;
  /* column_labels.border.bottom.style */
  border-bottom-width: 2px;
  /* column_labels.border.bottom.width */
  border-bottom-color: #D3D3D3;
  /* column_labels.border.bottom.color */
  border-left-style: none;
  /* column_labels.border.lr.style */
  border-left-width: 1px;
  /* column_labels.border.lr.width */
  border-left-color: #D3D3D3;
  /* column_labels.border.lr.color */
  border-right-style: none;
  /* column_labels.border.lr.style */
  border-right-width: 1px;
  /* column_labels.border.lr.width */
  border-right-color: #D3D3D3;
  /* column_labels.border.lr.color */
}

#auhridjtrm .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  /* column_labels.background.color */
  font-size: 100%;
  /* column_labels.font.size */
  font-weight: normal;
  /* column_labels.font.weight */
  text-transform: inherit;
  /* column_labels.text_transform */
  vertical-align: middle;
  padding: 5px;
  margin: 10px;
  overflow-x: hidden;
}

#auhridjtrm .gt_sep_right {
  border-right: 5px solid #FFFFFF;
}

#auhridjtrm .gt_group_heading {
  padding: 8px;
  /* row_group.padding */
  color: #333333;
  background-color: #FFFFFF;
  /* row_group.background.color */
  font-size: 100%;
  /* row_group.font.size */
  font-weight: initial;
  /* row_group.font.weight */
  text-transform: inherit;
  /* row_group.text_transform */
  border-top-style: solid;
  /* row_group.border.top.style */
  border-top-width: 2px;
  /* row_group.border.top.width */
  border-top-color: #D3D3D3;
  /* row_group.border.top.color */
  border-bottom-style: solid;
  /* row_group.border.bottom.style */
  border-bottom-width: 2px;
  /* row_group.border.bottom.width */
  border-bottom-color: #D3D3D3;
  /* row_group.border.bottom.color */
  border-left-style: none;
  /* row_group.border.left.style */
  border-left-width: 1px;
  /* row_group.border.left.width */
  border-left-color: #D3D3D3;
  /* row_group.border.left.color */
  border-right-style: none;
  /* row_group.border.right.style */
  border-right-width: 1px;
  /* row_group.border.right.width */
  border-right-color: #D3D3D3;
  /* row_group.border.right.color */
  vertical-align: middle;
}

#auhridjtrm .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  /* row_group.background.color */
  font-size: 100%;
  /* row_group.font.size */
  font-weight: initial;
  /* row_group.font.weight */
  border-top-style: solid;
  /* row_group.border.top.style */
  border-top-width: 2px;
  /* row_group.border.top.width */
  border-top-color: #D3D3D3;
  /* row_group.border.top.color */
  border-bottom-style: solid;
  /* row_group.border.bottom.style */
  border-bottom-width: 2px;
  /* row_group.border.bottom.width */
  border-bottom-color: #D3D3D3;
  /* row_group.border.bottom.color */
  vertical-align: middle;
}

#auhridjtrm .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
  /* row.striping.background_color */
}

#auhridjtrm .gt_from_md > :first-child {
  margin-top: 0;
}

#auhridjtrm .gt_from_md > :last-child {
  margin-bottom: 0;
}

#auhridjtrm .gt_row {
  padding-top: 8px;
  /* data_row.padding */
  padding-bottom: 8px;
  /* data_row.padding */
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  /* table_body.hlines.style */
  border-top-width: 1px;
  /* table_body.hlines.width */
  border-top-color: #D3D3D3;
  /* table_body.hlines.color */
  border-left-style: none;
  /* table_body.vlines.style */
  border-left-width: 1px;
  /* table_body.vlines.width */
  border-left-color: #D3D3D3;
  /* table_body.vlines.color */
  border-right-style: none;
  /* table_body.vlines.style */
  border-right-width: 1px;
  /* table_body.vlines.width */
  border-right-color: #D3D3D3;
  /* table_body.vlines.color */
  vertical-align: middle;
  overflow-x: hidden;
}

#auhridjtrm .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  /* stub.background.color */
  font-weight: initial;
  /* stub.font.weight */
  text-transform: inherit;
  /* stub.text_transform */
  border-right-style: solid;
  /* stub.border.style */
  border-right-width: 2px;
  /* stub.border.width */
  border-right-color: #D3D3D3;
  /* stub.border.color */
  padding-left: 12px;
}

#auhridjtrm .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  /* summary_row.background.color */
  text-transform: inherit;
  /* summary_row.text_transform */
  padding-top: 8px;
  /* summary_row.padding */
  padding-bottom: 8px;
  /* summary_row.padding */
  padding-left: 5px;
  padding-right: 5px;
}

#auhridjtrm .gt_first_summary_row {
  padding-top: 8px;
  /* summary_row.padding */
  padding-bottom: 8px;
  /* summary_row.padding */
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: solid;
  /* summary_row.border.style */
  border-top-width: 2px;
  /* summary_row.border.width */
  border-top-color: #D3D3D3;
  /* summary_row.border.color */
}

#auhridjtrm .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  /* grand_summary_row.background.color */
  text-transform: inherit;
  /* grand_summary_row.text_transform */
  padding-top: 8px;
  /* grand_summary_row.padding */
  padding-bottom: 8px;
  /* grand_summary_row.padding */
  padding-left: 5px;
  padding-right: 5px;
}

#auhridjtrm .gt_first_grand_summary_row {
  padding-top: 8px;
  /* grand_summary_row.padding */
  padding-bottom: 8px;
  /* grand_summary_row.padding */
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  /* grand_summary_row.border.style */
  border-top-width: 6px;
  /* grand_summary_row.border.width */
  border-top-color: #D3D3D3;
  /* grand_summary_row.border.color */
}

#auhridjtrm .gt_table_body {
  border-top-style: solid;
  /* table_body.border.top.style */
  border-top-width: 2px;
  /* table_body.border.top.width */
  border-top-color: #D3D3D3;
  /* table_body.border.top.color */
  border-bottom-style: solid;
  /* table_body.border.bottom.style */
  border-bottom-width: 2px;
  /* table_body.border.bottom.width */
  border-bottom-color: #D3D3D3;
  /* table_body.border.bottom.color */
}

#auhridjtrm .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  /* footnotes.background.color */
  border-bottom-style: none;
  /* footnotes.border.bottom.style */
  border-bottom-width: 2px;
  /* footnotes.border.bottom.width */
  border-bottom-color: #D3D3D3;
  /* footnotes.border.bottom.color */
  border-left-style: none;
  /* footnotes.border.lr.color */
  border-left-width: 2px;
  /* footnotes.border.lr.color */
  border-left-color: #D3D3D3;
  /* footnotes.border.lr.color */
  border-right-style: none;
  /* footnotes.border.lr.color */
  border-right-width: 2px;
  /* footnotes.border.lr.color */
  border-right-color: #D3D3D3;
  /* footnotes.border.lr.color */
}

#auhridjtrm .gt_footnote {
  margin: 0px;
  font-size: 90%;
  /* footnotes.font.size */
  padding: 4px;
  /* footnotes.padding */
}

#auhridjtrm .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  /* source_notes.background.color */
  border-bottom-style: none;
  /* source_notes.border.bottom.style */
  border-bottom-width: 2px;
  /* source_notes.border.bottom.width */
  border-bottom-color: #D3D3D3;
  /* source_notes.border.bottom.color */
  border-left-style: none;
  /* source_notes.border.lr.style */
  border-left-width: 2px;
  /* source_notes.border.lr.style */
  border-left-color: #D3D3D3;
  /* source_notes.border.lr.style */
  border-right-style: none;
  /* source_notes.border.lr.style */
  border-right-width: 2px;
  /* source_notes.border.lr.style */
  border-right-color: #D3D3D3;
  /* source_notes.border.lr.style */
}

#auhridjtrm .gt_sourcenote {
  font-size: 90%;
  /* source_notes.font.size */
  padding: 4px;
  /* source_notes.padding */
}

#auhridjtrm .gt_left {
  text-align: left;
}

#auhridjtrm .gt_center {
  text-align: center;
}

#auhridjtrm .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#auhridjtrm .gt_font_normal {
  font-weight: normal;
}

#auhridjtrm .gt_font_bold {
  font-weight: bold;
}

#auhridjtrm .gt_font_italic {
  font-style: italic;
}

#auhridjtrm .gt_super {
  font-size: 65%;
}

#auhridjtrm .gt_footnote_marks {
  font-style: italic;
  font-size: 65%;
}
</style>
<div id="auhridjtrm" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;"><table class="gt_table">
  <thead class="gt_header">
    <tr>
      <th colspan="5" class="gt_heading gt_title gt_font_normal gt_center" style>German Indefinite Articles (a/an)</th>
    </tr>
    <tr>
      <th colspan="5" class="gt_heading gt_subtitle gt_font_normal gt_center gt_bottom_border" style></th>
    </tr>
  </thead>
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1"></th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1">Masculine</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1">Feminine</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1">Neuter</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1">Plural</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr>
      <td class="gt_row gt_left gt_stub">Nominative</td>
      <td class="gt_row gt_left">ein</td>
      <td class="gt_row gt_left">eine</td>
      <td class="gt_row gt_left">ein</td>
      <td class="gt_row gt_left">keine</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Accusative</td>
      <td class="gt_row gt_left gt_striped">einen</td>
      <td class="gt_row gt_left gt_striped">eine</td>
      <td class="gt_row gt_left gt_striped">ein</td>
      <td class="gt_row gt_left gt_striped">keine</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Dative</td>
      <td class="gt_row gt_left">einem</td>
      <td class="gt_row gt_left">einer</td>
      <td class="gt_row gt_left">einem</td>
      <td class="gt_row gt_left">keinen</td>
    </tr>
    <tr>
      <td class="gt_row gt_left gt_stub">Genitive</td>
      <td class="gt_row gt_left gt_striped">eines</td>
      <td class="gt_row gt_left gt_striped">einer</td>
      <td class="gt_row gt_left gt_striped">eines</td>
      <td class="gt_row gt_left gt_striped">keiner</td>
    </tr>
  </tbody>
  
  
</table></div><!--/html_preserve-->


Building lists of stop words in Chinese has been done both manually and automatically [@Zou2006ACC] but so far none has been accepted as a standard [@Zou2006]. A full discussion of stop word identification in Chinese text would be out of scope for this book, so we will just highlight some of the challenges that differentiate it from English. 

<div class="rmdwarning">
<p>Chinese text is much more complex than portrayed here. With different systems and billions of users, there is much we won't be able to touch on here.</p>
</div>

The main difference from English is the use of logograms instead of letters to convey information. However, Chinese characters should not be confused with Chinese words. The majority of words in modern Chinese are composed of multiple characters. This means that inferring the presence of words is more complicated and the notion of stop words will affect how this segmentation of characters is done.

## Summary

TODO