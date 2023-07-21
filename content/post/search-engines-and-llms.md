---
title: "Search Engine Design in the Age of LLMs"
date: 2023-07-21T00:57:09-04:00
draft: false
tags: []
categories: []
---



# Search Engine Design in the Age of LLMs

### How paywalled information can be discovered by search engines without revealing the entire contents

“LLMs will change everything”, or more broadly, [AI will save the world](https://a16z.com/2023/06/06/ai-will-save-the-world/). Naturally, this has caused a lot of interest in LLMs. Contrast this with interest received by embeddings, which are a crucial ingredient for their working.

![img](/home/altairmn/Projects/altairmn.github.io/static/images/trends.png)Using the Roy Keyes definition of embeddings

- `*Embeddings are learned transformations to make data more useful*`

There’s a more detailed exposition later in the article if this definition is cryptic for you.

Why embeddings and what is their role in search engine design? Before we dive deeper into this question, we must start with understanding how search engines work.

## How search Engines work right now?

Search engines have perpetually running programs called crawlers that explore the web by following links. They’d process a page, then process all the links on that page, and so on. For each webpage, search engines have an entry in a table called “index”. Depending on the sophistication of the search engine, this index can have the URL of the page, keywords, freshness, number of links to that page, etc.

The index is the data structure that search engines look through when you make a search query. Since the goal of a search engine is to surface relevant results ranked to answer your query immediately, user specific information is also stored to substantiate the results.

We don’t need to dive into how sophisticated search engines could get. Just know that for every indexed page, search engines **necessarily** store some keywords, features, etc. that is used to assess the relevance of that page for a search query.

## Interplay of Search Engines and Paywalls

For the purpose of this article, we’d focus specifically on online news publications as the data under search.

Often times, page owners (i.e. webmasters) want their website/pages discovered without revealing the contents of the page. A well known example of this is paywalls on news websites. In order to be discoverable by a search engine, they have their content accessible to crawlers. But this opens them up to clever hacks which allow access to pages without paying.

To clearly elucidate how search engine design might change, we’ll focus solely on news articles. This is easily generalized.

## The New Search Engine

Consider publications like New York Times (NYT), The Washington Post (WaPo), and Wall Street Journal (WSJ). These are news publications and they put their articles behind a paywall. These allows they make a revenue from restricting access.

Let me describe the steps on how they’ll appear in search results in the new design.

## Step 1: Generate Embeddings of Articles

![img](/home/altairmn/Projects/altairmn.github.io/static/images/create-embeds.png)You might ask, what are ***embeddings***?

>  **Embedding**
>
> For our purpose, the word embedding means a representation of a piece of text as a sequence of numbers. This sequence of numbers is not arbitrary. It is designed so that embeddings of similar pieces of text are “closer” to each other than of dissimilar pieces. They can satisfy many other properties as well. You can learn more about embeddings here (OpenAI)  https://openai.com/blog/introducing-text-and-code-embeddings

> **Embedding Models**
>
> An embedding model is a “function” that converts a piece of text to a sequence of numbers. There are several embedding models that are out in the wild. Different embedding models are often designed to satisfy different use cases. `text-similarity-curie-001` is an embedding model designed for clustering, while `code-search-ada-001` is designed for code inputs. These are examples of models by OpenAI, but there are several [open source examples](https://github.com/Hironsan/awesome-embedding-models) as well. Note that the embeddings will look different if the embedding models are changed even if the input text is held constant.
>
> ![img](/home/altairmn/Projects/altairmn.github.io/static/images/different-embed-models.png)
> The above embeddings are only illustrative. We show how running a piece of text from a news article through different models leads to different embeddings.

We show how NYT generates embeddings for all it’s articles in the above example:

1. Label all articles from 1 → 100K (assuming they have 100K articles)
2. Chunk each article into several pieces of text. For example, let article 102 be ***[A Year of Cosmic Wonder With the James Webb Space Telescope](https://www.nytimes.com/2023/07/12/science/nasa-webb-telescope-one-year-anniversary.html)*** and the 5th chunk of the article is “…Jane Rigby, the senior project scientist for the telescope…”. Let this chunk be labeled as `(NYT, 102, 5)`. We’ll use this convention to label chunks.
3. Each chunk is sent to an embedding service to generate an embedding for that chunk. For example, for `(NYT, 102, 5)` we call the chunk `embed_(NYT, 102,5)`

# Step 2: Embedding Aggregation

Embeddings created by all the publications will now be aggregated. They’d send the embeddings to a hosted *Aggregator Service (AS)* that’ll store all the embeddings in the `AggEmbedStore` database. This is crucial to enable search over all the publications.

![img](/home/altairmn/Projects/altairmn.github.io/static/images/agg-embed.png)

In practice, web services of these publications will continually be sending the embeddings to the *AS,* along with labels describing the embeddings.

# Step 3: Search by User

The user search experience is pretty straightforward.

1. User enters query, for instance, “middle east war”
2. Query is converted to an embedding *e* using an embedding model. Let’s assume `text-similarity-curie-001`
3. The embedding **e** is used to search against all embeddings in `AggEmbedStore` to find similar embeddings. The search engine can find top 10 similar embeddings using [cosine similarity](https://en.wikipedia.org/wiki/Cosine_similarity).
4. The similar embeddings are sent out to servers of the publication(s) to retrieve the associated chunk. For ex: if `embed_(WSJ, 12, 3)` is in the results, then the associated `(WSJ, 12, 3)` chunk is retrieved.
5. List of results composed of relevant chunk + link to the associated article is compiled and sent to the user.

![img](/home/altairmn/Projects/altairmn.github.io/static/images/search.png)

# Conclusion

As we can see, search results rely solely on embeddings.  Therefore, the articles don’t have to be “open to the web” for crawlers to access them.

I doubt that this model will be adopted by news publications and search engines, but it’s likely that a search functionality over proprietary data from multiple sources could use something like this. Furthermore, when results are surfaced, the user doesn’t have to be shown the relevant chunk. They could just receive a blurb of the associated article. This way, even the chunks don’t have to be revealed to the search engine.

# FAQs

**Q. What are the tech infrastructure pieces required for this?**

In practice, a company would provide this service. They’d setup an embedding service, and the `AggEmbedStore` which would create and store the embeddings. They’d also give a search endpoint which will take in a search query and return the relevant search results. A search engine can ping the search endpoint to retrieve the search results.

**Q. Why is this approach better than the existing approach?**

This new approach is better because it solves a problem inherent in the current system where crawlers cannot access some parts of the web due to firewalls, passwords, paywalls, or direct instructions not to index. By using embeddings and a centralized aggregator service, we can maintain the discoverability of articles even behind paywalls without revealing the contents of the article to search engines, etc.

We can add ‘premium articles’ to the search results. This will show the relevant chunk to the user, and the user can pay to access the whole article.

![img](/home/altairmn/Projects/altairmn.github.io/static/images/search-with-paywall.png)

**Q. What are the specifics of how the search works?**

Refer to [this OpenAI article](https://openai.com/blog/introducing-text-and-code-embeddings) on embeddings to learn more about generation of embeddings and similarity search.