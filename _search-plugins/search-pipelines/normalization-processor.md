---
layout: default
title: Normalization
nav_order: 70
has_children: false
parent: Search processors
grand_parent: Search pipelines
---

# Normalization processor
Introduced 2.10
{: .label .label-purple }

The `normalization-processor` is a search phase results processor that runs between the query and fetch phases of search execution. It intercepts the query phase results and then normalizes and combines the document scores from different query clauses before passing the documents to the fetch phase.

## Score normalization and combination

Many applications require both keyword matching and semantic understanding. For example, BM25 accurately provides relevant search results for a query containing keywords, and neural networks perform well when a query requires natural language understanding. Thus, you might want to combine BM25 search results with the results of a k-NN or neural search. However, BM25 and k-NN search use different scales to calculate relevance scores for the matching documents. Before combining the scores from multiple queries, it is beneficial to normalize them so that they are on the same scale, as shown by experimental data. For further reading about score normalization and combination, including benchmarks and various techniques, see [this semantic search blog post](https://opensearch.org/blog/semantic-science-benchmarks/).

## Query then fetch

OpenSearch supports two search types: `query_then_fetch` and `dfs_query_then_fetch`. The following diagram outlines the query-then-fetch process, which includes a normalization processor.

![Normalization processor flow diagram]({{site.url}}{{site.baseurl}}/images/normalization-processor.png)

When you send a search request to a node, the node becomes a _coordinating node_. During the first phase of search, the _query phase_, the coordinating node routes the search request to all shards in the index, including primary and replica shards. Each shard then runs the search query locally and returns metadata about the matching documents, which includes their document IDs and relevance scores. The `normalization-processor` then normalizes and combines scores from different query clauses. The coordinating node merges and sorts the local lists of results, compiling a global list of top documents that match the query. After that, search execution enters a _fetch phase_, in which the coordinating node requests the documents in the global list from the shards where they reside. Each shard returns the documents' `_source` to the coordinating node. Finally, the coordinating node sends a search response containing the results back to you.

## Request body fields

The following table lists all available request fields.

Field | Data type | Description
:--- | :--- | :---
`normalization.technique` | String | The technique for normalizing scores. Valid values are [`min_max`](https://en.wikipedia.org/wiki/Feature_scaling#Rescaling_(min-max_normalization)), [`l2`](https://en.wikipedia.org/wiki/Cosine_similarity#L2-normalized_Euclidean_distance), and [`z_score`](https://en.wikipedia.org/wiki/Standard_score). Optional. Default is `min_max`.
 `normalization.parameters.lower_bounds` | Array of objects | Defines the lower bound values (the minimum threshold scores) for each query. The array must contain the same number of objects as the number of queries. Optional. Applies only when the normalization technique is [`min_max`](https://en.wikipedia.org/wiki/Feature_scaling#Rescaling_(min-max_normalization)). If not provided, OpenSearch does not apply a lower bound to any subquery and uses the actual minimum score from the retrieved results for normalization.
`normalization.parameters.lower_bounds.mode` | String | Specifies how the lower bound is applied to a query. Valid values are: <br> - `apply`: Uses `min_score` for normalization without modifying the original scores. Formula: `min_max_score = if (score < lowerBoundScore) then (score - minScore) / (maxScore - minScore) else (score - lowerBoundScore) / (maxScore - lowerBoundScore)`. <br> - `clip`: Replaces scores below the lower bound with `min_score`. Formula: `min_max_score = if (score < lowerBoundScore) then 0.0 else (score - lowerBoundScore) / (maxScore - lowerBoundScore)`. <br> - `ignore`: Does not apply a lower bound to this query and uses the standard `min_max` formula instead. <br> Optional. Default is `apply`. 
`normalization.parameters.lower_bounds.min_score` | Float | The lower bound threshold. Valid values are in the [-10000.0, 10000.0] range. If `mode` is set to `ignore`, then this value has no effect. Optional. Default is `0.0`.
`normalization.parameters.upper_bounds` | Array of objects | Defines the upper bound values (the maximum threshold scores) for each query. The array must contain the same number of objects as the number of queries. Optional. Applies only when the `normalization.technique` is set to `min_max`. If not provided, OpenSearch does not apply an upper bound to any subquery and uses the actual maximum score from the retrieved results for normalization.
`normalization.parameters.upper_bounds.mode` | String | Specifies how the upper bound is applied to a query. Valid values are: <br> - `apply`: Uses `max_score` for normalization without modifying the original scores. Formula: `min_max_score = if (score > upperBoundScore) then (score - minScore) / (maxScore - minScore) else (score - minScore) / (upperBoundScore - minScore)`. <br> - `clip`: Replaces scores above the upper bound with `max_score`. Formula: `min_max_score = if (score > upperBoundScore) then 1.0 else (score - minScore) / (upperBoundScore - minScore)`. <br> - `ignore`: Does not apply an upper bound to this query and uses the standard `min_max` formula instead. <br> Optional. Default is `apply`. 
`normalization.parameters.upper_bounds.max_score` | Float | The upper bound threshold. Valid values are in the [-10000.0, 10000.0] range. If `mode` is set to `ignore`, then this value has no effect. Optional. Default is `1.0`. 
`combination.technique` | String | The technique for combining scores. Valid values are [`arithmetic_mean`](https://en.wikipedia.org/wiki/Arithmetic_mean), [`geometric_mean`](https://en.wikipedia.org/wiki/Geometric_mean), and [`harmonic_mean`](https://en.wikipedia.org/wiki/Harmonic_mean). Optional. Default is `arithmetic_mean`. `z_score` supports only `arithmetic_mean`.
`combination.parameters.weights` | Array of floating-point values | Specifies the weights to use for each query. Valid values are in the [0.0, 1.0] range and signify decimal percentages. The closer the weight is to 1.0, the more weight is given to a query. The number of values in the `weights` array must equal the number of queries. The sum of the values in the array must equal 1.0. Optional. If not provided, all queries are given equal weight.
`tag` | String | The processor's identifier. Optional.
`description` | String | A description of the processor. Optional.
`ignore_failure` | Boolean | For this processor, this value is ignored. If the processor fails, the pipeline always fails and returns an error. 

## Example 

The following example demonstrates using a search pipeline with a `normalization-processor`. 

For a comprehensive example, follow the [Getting started with semantic and hybrid search]({{site.url}}{{site.baseurl}}/ml-commons-plugin/semantic-search#tutorial).

### Creating a search pipeline 

The following request creates a search pipeline containing a `normalization-processor` that uses the `min_max` normalization technique and the `arithmetic_mean` combination technique. The combination technique assigns a weight of 30% to the first query and 70% to the second query:

```json
PUT /_search/pipeline/nlp-search-pipeline
{
  "description": "Post processor for hybrid search",
  "phase_results_processors": [
    {
      "normalization-processor": {
        "normalization": {
          "technique": "min_max"
        },
        "combination": {
          "technique": "arithmetic_mean",
          "parameters": {
            "weights": [
              0.3,
              0.7
            ]
          }
        }
      }
    }
  ]
}
```
{% include copy-curl.html %}

The following example demonstrates using the `lower_bounds` and `upper_bounds` parameters with the `min_max` normalization technique. It omits the `weights` parameter in the combination technique, causing the queries to be weighted equally by default. In this example, the `lower_bounds` parameter is used to set different lower bounds for each query in a hybrid search, and the `upper_bounds` parameter is used to set different upper bounds. For the first query, a lower bound of 0.5 is applied and an upper bound of 0.8 is clipped. For the second query, both the lower bound and the upper bound are ignored. This allows for fine-tuning of the normalization process for each individual query in a hybrid search:

```json
PUT /_search/pipeline/nlp-search-pipeline
{
  "description": "Post processor for hybrid search",
  "phase_results_processors": [
    {
      "normalization-processor": {
        "normalization": {
          "technique": "min_max",
          "parameters": {
            "lower_bounds": [
                {
                  "mode": "apply",
                  "min_score": 0.5
                },
                {
                  "mode": "ignore"
                }
              ],
            "upper_bounds": [
              {
                "mode": "clip",
                "max_score": 0.8
              },
              {
                "mode": "ignore"
              }
            ]
          }
        },
        "combination": {
          "technique": "arithmetic_mean"
        }
      }
    }
  ]
}
```
{% include copy-curl.html %}

### Using a search pipeline

Provide the query clauses that you want to combine in a `hybrid` query and apply the search pipeline created in the previous section so that the scores are combined using the chosen techniques:

```json
GET /my-nlp-index/_search?search_pipeline=nlp-search-pipeline
{
  "_source": {
    "exclude": [
      "passage_embedding"
    ]
  },
  "query": {
    "hybrid": {
      "queries": [
        {
          "match": {
            "text": {
              "query": "horse"
            }
          }
        },
        {
          "neural": {
            "passage_embedding": {
              "query_text": "wild west",
              "model_id": "aVeif4oB5Vm0Tdw8zYO2",
              "k": 5
            }
          }
        }
      ]
    }
  }
}
```
{% include copy-curl.html %}

For more information about setting up hybrid search, see [Hybrid search]({{site.url}}{{site.baseurl}}/search-plugins/hybrid-search/).

## Search tuning recommendations

To improve search relevance, we recommend increasing the sample size.

If the hybrid query does not return some expected results, it may be because the subqueries return too few documents. The `normalization-processor` only transforms the results returned by each subquery; it does not perform any additional sampling. During our experiments, we used [nDCG@10](https://en.wikipedia.org/wiki/Discounted_cumulative_gain) to measure quality of information retrieval depending on the number of documents returned (the size). We have found that a size in the [100, 200] range works best for datasets of up to 10M documents. We do not recommend increasing the size beyond the recommended values because higher size values do not improve search relevance but increase search latency.
