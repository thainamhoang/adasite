---
layout: default
---
<p>
    <img src="assets/banner.jpg" alt="Banner" class="banner">
</p>

# Creative breakthroughs occur, when worlds collide

*Thai-Nam Hoang, Valentin Peyron, Paul-Bogdan Jurcut, Quentin Esteban, Jan Kokla*

In 2004, American entrepreneur Frans Johansson published the book
[“The Medici Effect: Breakthrough Insights at the Intersection of
Ideas, Concepts, and Cultures”](https://www.goodreads.com/pt/book/show/20482413). 
In other words, by merging ideas from a range of diverse backgrounds,
one can increase the likelihood of intellectual cross-pollination,
which might lead to innovation and success. Our goal is to see if this 
holds true in the movie industry.

## "At the intersection"?

Before we can dive deep into the movies, let’s first think about the 
concept of “being at the intersection” for a second. Most natural way 
of looking at it is network graphs. With 2 simple examples, we will 
introduce the two measures that we’re going to use 
for quantifying it: **_degree_** and **_betweenness_**.

### Betweenness

If you take a look at the graph below, you see that we have 2 bigger 
clusters and one node that sits at the intersection. 
This is exactly what the Medici effect referred to! The yellow node in the middle 
is a combination of the ideas from both clusters or controls the flow of information 
and should be more successful. Visually it all looks 
really simple to grasp right, but how can we quantify it? This is where 
**betweenness** measure comes in.

{% include theory/betweenness.html %}

If you take all pairs of nodes from the graph and find the shortest paths 
between them, then betweenness centrality for a certain node is _the percentage 
of these shortest paths that go through the node_. If you now once again look 
at the graph above and focus on the colors of the nodes [^1], you see 
that the lighter the color, the bigger the betweenness. We have 
successfully quantified “being at the intersection” for clustered network graphs.

[^1]: You can hover over the nodes to see the numerical measure.

### Degree

What if we don’t have such clear clusters, but rather have a big chunk of quite 
similar movies and then some outliers as seen from the graph below? We don’t really 
have nodes that act as bridges between clusters and sit at the intersections. In 
that case, let’s redefine the Medici effect in the movie industry a bit and say 
that the most successful movies will be <u>the ones that have taken ideas from many 
other movies and thus are connected to the biggest possible number of other nodes</u>. 
You might already have guessed… degree of the node is exactly what we need.

{% include theory/degree.html %}

The degree of a node is the number of connections that it has to other nodes in the 
network. This time, the color of the node reflects exactly that 
(lighter color 
means higher degree). If you hover over the nodes, you can compare the betweenness 
and degree measures for the node. Naturally, the ones in the big cluster have higher 
degree, compared to the ones in the outskirts.

## Data

Now that we have the theory settled, let's look at the data. We used 
[CMU Movie Summary Corpus](https://www.cs.cmu.edu/~ark/personas/) 
and merged it with the 
[IMDB ratings](https://developer.imdb.com/non-commercial-datasets/) as we needed to 
quantify the success of the movie.

{% include analysis/year_histogram.html %}

From the histogram above, we can see that there are quite a big amount of movies. 
To cope with such quantities, we decided to run the analysis on decade basis. 
In other words all the correlation analyses below are done for 10 years: 1920-1929, 
1930-1939 etc. As visualising even 1000 nodes would be computationally heavy, all 
the following visualisations of network graphs use year 2012 as an example (388 nodes).

...

## What about the movies?

We have found ways for quantifying the centrality of the node, but how about movies? 
How can we generate a graph based on the data at hand? For that we came up with two 
alternative approaches, which include **embeddings** and **classification**.

### Embeddings

The idea is simple: find the similarity between all possible combinations of movies 
and if they're similar enough, create an edge. The reality is a bit more complex.
We first need to turn movies into vector representations. In order to do that, 
we'll use an embedding model 
[E5-large-v2](https://huggingface.co/intfloat/e5-large-v2) that takes the plot of 
the movie as an input and converts it into normalized feature vector with 
embedding size of 1024.

Once we have vectorized the plots and as they're normalized, we can calculate 
the cosine similarities between the plots with dot product. Below 
you can see an example of similarity matrix with 20 random movies. Notice that 
one of the quirks of the embedding model is that all the cosine similarities are
between 0.7 and 1.0. As we still have a distribution, it will not be a problem for
us.

{% include analysis/similarity_matrix.html %}

Let's make a sanity check and take a look at the similarity matrix to see 
if it intuitively makes sense.
[_Sortie 67_](https://www.imdb.com/title/tt1651916/) and 
[_Victim_](https://s.media-imdb.com/title/tt1684564/?ref_=nm_flmg_t_1_msdp) 
both talk about witnessing a brutal murder, so it makes sense to qualify them as 
similar. On the other hand, 
[_How to Start a Revolution_](https://www.imdb.com/title/tt1956516/) is not
too similar to any other movie. This is expected as it's a biographical 
documentary, whereas all the others are based on fiction. It is especially different 
from [_Rabbit Hole_](https://www.imdb.com/title/tt0935075/?ref_=nv_sr_srsg_3_tt_7_nm_1_q_rabbit%2520h),
which talks about a happy couple whose life is turned upside down after their young 
son dies in an accident. Thus, it seems that the embedding model has bone a decent 
job.

Once we have the similarity matrix, we need to figure out the **threshold** for the 
similarities. We want to create an edge between all the movies for which the 
similarity is higher than the threshold. [_How to Start a Revolution_](https://www.imdb.com/title/tt1956516/)
from the similarity matrix, for example, will not have too many connected movies. 
In order to come up with threshold, let's look at the histogram of the similarities below.

{% include analysis/similarity_hist.html %}

The values are normally distributed and in order to keep it simple, let's set the limit to 75% percentile. 
This way we will have enough connected nodes, but not too much. We have finally reached the goal... we have the graph. 
For better visibility, the one below only includes the movies from 2012. As you 
can see, we have only one big cluster with some movies on the outskirts. Thus, we 
used degree to color the nodes (<u>the lighter the color, the higher the degree</u>). 
The size of the node is dependent on the IMDB rating of the movie (<u>the bigger 
the node, the higher the rating</u>). You can see the 
neighbouring movies explicitly, when you click on one of the nodes.

<div class="noteBoxes note" style="display:flex;">
	<img src="assets/light_bulb.svg" class="lightbulb" style="margin-bottom:auto;">
	 <div>
		<p style="margin-bottom: 0">Hover over the nodes to see attributes and click on it to see its neighbours.</p>
        <p style="margin-bottom: 0">Feel free to drag, zoom and filter based on IMDB rating.</p>
	</div>
</div>


{% include analysis/2013_embedding_example.html %}

As we have only one big cluster, then the question to ask when coming back to 
the the research problem is "whether the movies with higher degree are more successful", 
Thus, we should see that the lighter nodes are bigger. From the visual inspection 
it is really hard to tell if there is any association. There are bigger nodes 
(higher rated → more successful) 
at the outskirts as well as in the middle of the cluster. It might even be that 
the bigger clusters tend to be at the outskirts. To get a quantitative answer
to our research question, we will perform **correlation analysis** (see further
below).

### Classification

Before we get to correlations, let's see another approach for generating network 
graphs.

#### Genres

TBD

#### Graph Generation

TBD

## Correlation Analysis

### Simple Correlation

TBD

#### Embedding Approach

TBD

####  Classification Approach

TBD

### Partial Correlations

TBD

#### Embedding Approach

TBD

####  Classification Approach

TBD


## Ethical Risks

Overall, our project does not seem to have many ethical risks. Indeed, as the project aims to determine if the best movies are at the "Intersection," the worst outcome could be inaccurate results. However, let's conduct a more in-depth analysis of our project from an ethical risk standpoint to identify potential risks and explore mitigation strategies. 
PS: This analysis was conducted at the end of our work, so none of the proposed measures have been implemented yet.

#### Dataset 
Firstly, let's examine the dataset. It was created by a team at Carnegie Mellon University and consists of 42,306 movie plot summaries extracted from Wikipedia, along with aligned metadata from Freebase. Therefore, the data were sourced from public domains. We supplemented this dataset with IMDb ratings and vote counts from a public database. Data gaps were not addressed in our study.

#### Beneficence
Our project offers valuable insights into cinematic trends and contributes to understanding the factors behind movie success. Specifically, it explores whether being at the "Intersection" is beneficial for a film.

#### Non-Maleficence
1. Risks: Incorrect results from our study could potentially lead to misguided decisions in movie production, affecting ratings and possibly the financial success of the involved parties. However, since our analysis is based solely on ratings, financial implications are uncertain.
2. Mitigation: To reduce these risks, we could cross-reference our data with another dataset to minimize misinformation. Performing multiple analyses with the same goal and comparing them could also be beneficial. We did this by using various metrics to define "being at the Intersection."

#### Privacy
1. Risks: As the data used are completely anonymous, there are no privacy concerns with our dataset.
2. Mitigation: Since our dataset is derived from anonymous data, no additional privacy measures are required.

#### Fairness
1. Risks: From a human fairness perspective, our study did not use data related to actors or characters, thus eliminating human-related fairness risks. However, a minor fairness issue might arise from the genres of movies, as our analysis includes genre information, potentially leading to skewed results due to genre distribution imbalances.
2. Mitigation: To mitigate this risk, it would be useful to ensure a balanced representation of genres. We could group genres for comparative analysis.

#### Sustainability 
1. Risks: Our project does not require excessive computational power, posing no significant sustainability issues. However, maintaining an up-to-date dataset requires regular updates, which, while minimal in energy consumption, could be time-consuming. An annual update should suffice, with minimal environmental impact.

#### Empowerment
Our analysis is not specifically targeted at any particular group of people. As the dataset is anonymous and the results aim to identify a correlation between quality and being at the Intersection, there are no apparent risks, and hence no need for specific mitigation measures.


## Summary

TBD
