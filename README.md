# MovieRecommender
by Spiros Valouxis and Athanasios Georgoutsos
## Dataset
Our dataset is based on [Carnegie Mellon Movie Summary Corpus](http://www.cs.cmu.edu/~ark/personas/). It contains 22301 movie descriptions, which consist of the title, genre keywords and a synopsis of the plot. We work on a subset of 5000 movies.

## Part1: Recommendations Based on Content
<img src="http://clture.org/wp-content/uploads/2015/12/Netflix-Streaming-End-of-Year-Posts.jpg" width="70%">
Here, we develop a content-based movie recommender system (https://en.wikipedia.org/wiki/Recommender_system). The goal of a recommender system is to suggest a small number of objects from a larger collection to a user, which should be appealing to him, based on certain criteria. Movie reccomender systems are divided based on the filtering of the objects for suggestion. The two main categories are collaborative filtering, where the system suggests objects that have been evaluated positively by users with similar evaluation history with our user, and content based filtering, where the system suggests objects with similar content with the ones he has previously positively evaluated.

Our following movie recommender system is based **on the content of the movies**, and more specifically on their synopses (**corpus**). 

### Preprocessing

Here, we try to clear the movies' summaries from words that do not provide useful information regarding their plot. 

The Preprocessing phase consists of the following:


*   Remove words starting with upper case characters
*   Make all characters lower case
*   Remove stopwords
*   Remove punctuations
*   Remove short strings (1 or 2 characters)

### Convert to TFIDF

Converting corpus to a tf-idf representation.

Then, we optimize our `TfidfVectorizer` researching our `content_recommenders's` suggestions and changing the vectorizer's parameters in order to improve our recommender's performance. Since many transformations have already been applied in the preprocessing phase, here we mainly remove the most and least frequent words. This procedure removes a large portion of the unique words found in the summaries.

Because we could not check the new results for 5000 movies, we checked specific movie IDs, which are the ones that we use as examples for our observations in the following parts.

Another goal we had in mind was to reduce the dimensions of our vector, preparing for the next parts of the exercise, but not so much that the suggestions of our system become considerably worse. 

### Deep Learning: Creating Corpora Using Word Emmbeddings


The approach to construct our recommender based on a tfidf vectorizer has, as it can be observed, disadvanages. Here we will try to use **word embeddings** provided by the **Word2Vec model**.

Because our dataset is too small to produce our own embeddings we load some ready, pretrained embeddings (using the deep learning methodology called Transfer Learning).

In order to include the knowledge of the pretrained embeddings in our own corpus we do the following:

For each movie summary $d$, which consists of $N_d$ words $w_i$, each word's  $tfidf$ is:

$$ tfidf(w_i) = tf(w_i,d) \cdot idf(w_i)$$

Each word $w_i$ has a vector $W2V(w_i)$ from the embeddings model with dimension $m$. 

For each movie d, we can create a vector $W2V(d)$ with $m$ dimensions using $tfidf(w_i)$ as a weight for each embedding $W2V(w_i)$:

$$ W2V(d) = \frac{tfidf(w_1)\cdot W2V(w_i) + tfidf(w_2)\cdot W2V(w_2) + \dotsc  + tfidf(w_{N_{d}})\cdot W2V(w_{N_{d}})}{tfidf(w_1)+tfidf(w_2)+ \dotsc + tfidf(w_{N_{d}})}$$

### Results & Conclusions

#### Pros of a Tfidf recommender

*   Excellent performance. After a good preprocessing phase, which removes most of the "junk" words, a Tfidf recommender can help us find movies similar to our input easily, just by sorting the list based on the movie vectors' cosine similarity and returning the movies with the best scores.
*   Fast train and transform runtimes.
*   Easily constructed, with possible implementations with other sorting criteria other than cosine similarity, if needed.
*   Offers many helpful parameters. These include a wide variety of preprocessing transformations that may improve the system's results, depending on the problem.


#### Cons of a Tfidf recommender


*   Deals with each word individually, not considering its context, unlike word embeddings. This can result to movie recommendations, that contain similar words in their summaries, but belong to completely different movie genres. 
*   Takes into account every word that "survives" the preprocessing phase, even some semantically uninteresting words that don't contribute to finding relevant suggestions. It is important to mention that the subtraction of every useless word of the summaries is an impossible task (unless someone hand-picks the useless words!!!)


#### Pros of a Word2Vec recommender

Recommenders based on Word2Vec embeddings can include knowledge already acquired by huge datasets and help us interpret each word, not only individually, as we did before, but taking into account the word's context.

#### Cons of a Word2Vec recommender

*   One weakness is that, in order to acquire the knowledge of the huge dataset, it is obligatory to load the embeddings. This comes with a long loading time, that grows larger as the size of the external data that we load.

*   Also, they only contain words found in their vocabulary. This is probably not a problem for larger datasets, but for smaller ones it poses a challenge.

**On tested movie IDs, the suggestions, in our opinion, are worse for the embeddings than the ones for tf-idf.**

For specific movie recommendations and a deeper dive into the comparison between the TF-IDF and the Word2Vec recommenders, please see the notebook.





## Part2: Semantic Mapping of Data using SOM (Genre Clustering)
<img src="https://i.imgur.com/Z4FdurD.jpg" width="60%">

### Dataset Building

In part 2 we build a 2D grid based on the topological properties of Self Organizing Maps (SOM) where our movies will be depicted clustered based on their categories as well.

We choose as `my_best_corpus` the corpus that came from the optimized Tfidf vectorizer.

### Clustering

Typically, clustering at a SOM derives from the unified distance matrix (U-matrix): for each node its mean distance from its neighbors is being computed. 

Blue color at SOM means this distance is small.
Red color at SOM means this distance is big.

Therefore, the neurons of a cluster tend to be in a blue-ish colored region, while their borders with another cluster tend to be a red-ish colored region.

We group the neurons using the k-Means algorithm. We choose k=20 initially.

### SOM and Clustering Exploration 

- For good clustering, U-matrix should show clusters with blue-green colors and borders with red (as much as it is possible). Conclusions should be based on number of movies, number of neurons and quality of U-matrix.
- k parameter of k-Means should approximate the number of clusters of U-matrix. Small values for k do not respect the U-matrix borders, while large values tend to create sub-clusters. The latter is not that harmful, but requires more extensive analysis.
- Making observations for smaller maps and smaller final sets, then expanding them to larger maps and the whole movie set.
- Some topological characteristics are visible only for larger maps. Exploration sizes are 20x20, 25x25 and 30x30 with respective values for k.

### Finding optimal SOM

Below we experiment with different grid sizes and k's (number of neighbors in algorithm), we examine the topological properties of the produced SOMs and, finally, we choose the best SOM. We analyze every SOM using the same procedures from the above steps.

We begin the exploration with grid sizes 20x20, 25x25 and 30x30, similar k values and with just 2000 from our 5000 movies (40%). Then, when we have locked on some parameter values, we are going to use our whole dataset.

### SOM (k=25)
![Alt text](image-1.png)

### Cluster Analysis
For a deeper dive into the analysis of each cluster in the SOMs, please see the notebook.