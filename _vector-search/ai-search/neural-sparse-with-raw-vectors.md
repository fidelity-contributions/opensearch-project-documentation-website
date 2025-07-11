---
layout: default
title: Neural sparse search using raw vectors
parent: Neural sparse search
grand_parent: AI search
nav_order: 30
has_children: false
redirect_from:
  - /search-plugins/neural-sparse-with-raw-vectors/
---

# Neural sparse search using raw vectors

If you're using self-hosted sparse embedding models, you can ingest raw sparse vectors for use in neural sparse search. 

## Example

The following example ingests sparse vectors into an OpenSearch index and then uses a sparse vector to search for matching documents.

### Step 1: Create an index

To ingest documents containing raw sparse vectors, create a rank features index:

```json
PUT /my-nlp-index
{
  "mappings": {
    "properties": {
      "id": {
        "type": "text"
      },
      "passage_embedding": {
        "type": "rank_features"
      },
      "passage_text": {
        "type": "text"
      }
    }
  }
}
```
{% include copy-curl.html %}

### Step 2: Ingest documents into the index

To ingest documents into the index created in the previous step, send the following request:

```json
PUT /my-nlp-index/_doc/1
{
  "passage_text": "Hello world",
  "id": "s1",
  "passage_embedding": {
    "hi" : 4.338913,
    "planets" : 2.7755864,
    "planet" : 5.0969057,
    "mars" : 1.7405145,
    "earth" : 2.6087382,
    "hello" : 3.3210192
  }
}
```
{% include copy-curl.html %}

### Step 3: Search the data using a sparse vector

To search the documents using a sparse vector, provide the sparse embeddings in the `neural_sparse` query:

```json
GET my-nlp-index/_search
{
  "query": {
    "neural_sparse": {
      "passage_embedding": {
        "query_tokens": {
          "hi" : 4.338913,
          "planets" : 2.7755864,
          "planet" : 5.0969057,
          "mars" : 1.7405145,
          "earth" : 2.6087382,
          "hello" : 3.3210192
        }
      }
    }
  }
}
```
{% include copy-curl.html %}

## Accelerating neural sparse search

To learn more about improving retrieval time for neural sparse search, see [Accelerating neural sparse search]({{site.url}}{{site.baseurl}}/search-plugins/neural-sparse-search/#accelerating-neural-sparse-search).

## Next steps

- Explore our [tutorials]({{site.url}}{{site.baseurl}}/vector-search/tutorials/) to learn how to build AI search applications. 
