---
layout: default
title: Search
parent: Search APIs
nav_order: 10
redirect_from:
  - /opensearch/rest-api/search/
  - /api-reference/search/
---

# Search API
**Introduced 1.0**
{: .label .label-purple }

The search API operation lets you search your cluster for data.

## Endpoints

```json
GET /<index>/_search
GET /_search

POST /<index>/_search
POST /_search
```

## Query parameters

All parameters are optional.

Parameter | Type | Description
:--- | :--- | :---
allow_no_indices | Boolean | Whether to ignore wildcards that don’t match any indexes. Default is `true`.
allow_partial_search_results | Boolean | Whether to return partial results if the request runs into an error or times out. Default is `true`.
analyzer | String | Analyzer to use in the query string.
analyze_wildcard | Boolean | Whether the update operation should include wildcard and prefix queries in the analysis. Default is `false`.
batched_reduce_size | Integer | How many shard results to reduce on a node. Default is 512.
cancel_after_time_interval | Time | The time after which the search request will be canceled. Request-level parameter takes precedence over cancel_after_time_interval [cluster setting]({{site.url}}{{site.baseurl}}/api-reference/cluster-settings/). Default is -1.
ccs_minimize_roundtrips | Boolean | Whether to minimize roundtrips between a node and remote clusters. Default is `true`.
default_operator | String | Indicates whether the default operator for a string query should be AND or OR. Default is OR.
df | String | The default field in case a field prefix is not provided in the query string.
docvalue_fields | String | The fields that OpenSearch should return using their docvalue forms.
expand_wildcards | String | Specifies the type of index that wildcard expressions can match. Supports comma-separated values. Valid values are all (match any index), open (match open, non-hidden indexes), closed (match closed, non-hidden indexes), hidden (match hidden indexes), and none (deny wildcard expressions). Default is open.
explain | Boolean | Whether to return details about how OpenSearch computed the document's score. Default is `false`.
from | Integer | The starting index to search from. Default is 0.
ignore_throttled | Boolean | Whether to ignore concrete, expanded, or indexes with aliases if indexes are frozen. Default is `true`.
ignore_unavailable | Boolean | Specifies whether to include missing or closed indexes in the response and ignores unavailable shards during the search request. Default is `false`.
lenient | Boolean | Specifies whether OpenSearch should accept requests if queries have format errors (for example, querying a text field for an integer). Default is `false`.
max_concurrent_shard_requests | Integer | How many concurrent shard requests this request should execute on each node. Default is 5.
phase_took | Boolean | Whether to return phase-level `took` time values in the response. Default is `false`.
pre_filter_shard_size | Integer | A prefilter size threshold that triggers a prefilter operation if the request exceeds the threshold. Default is 128 shards.
preference | String | Specifies the shards or nodes on which OpenSearch should perform the search. For valid values, see [The `preference` query parameter](#the-preference-query-parameter). 
q | String | Lucene query string’s query.
request_cache | Boolean | Specifies whether OpenSearch should use the request cache. Default is whether it’s enabled in the index’s settings.
rest_total_hits_as_int | Boolean | Whether to return `hits.total` as an integer. Returns an object otherwise. Default is `false`.
routing | String | Value used to route the update by query operation to a specific shard.
scroll | Time | How long to keep the search context open.
search_type | String | Whether OpenSearch should use global term and document frequencies when calculating relevance scores. Valid choices are `query_then_fetch` and `dfs_query_then_fetch`. `query_then_fetch` scores documents using local term and document frequencies for the shard. It’s usually faster but less accurate. `dfs_query_then_fetch` scores documents using global term and document frequencies across all shards. It’s usually slower but more accurate. Default is `query_then_fetch`.
seq_no_primary_term | Boolean | Whether to return sequence number and primary term of the last operation of each document hit.
size | Integer | How many results to include in the response.
sort | List | A comma-separated list of &lt;field&gt; : &lt;direction&gt; pairs to sort by.
_source | String | Whether to include the `_source` field in the response.
_source_excludes | List | A comma-separated list of source fields to exclude from the response.
_source_includes | List | A comma-separated list of source fields to include in the response.
stats | String | Value to associate with the request for additional logging.
stored_fields | Boolean | Whether the get operation should retrieve fields stored in the index. Default is `false`.
suggest_field | String | Fields OpenSearch can use to look for similar terms.
suggest_mode | String | The mode to use when searching. Available options are `always` (use suggestions based on the provided terms), `popular` (use suggestions that have more occurrences), and `missing` (use suggestions for terms not in the index).
suggest_size | Integer | How many suggestions to return.
suggest_text | String | The source that suggestions should be based off of.
terminate_after | Integer | The maximum number of matching documents (hits) OpenSearch should process before terminating the request. Default is 0.
timeout | Time | How long the operation should wait for a response from active shards. Default is `1m`.
track_scores | Boolean | Whether to return document scores. Default is `false`.
track_total_hits | Boolean or Integer | Whether to return how many documents matched the query.
typed_keys | Boolean | Whether returned aggregations and suggested terms should include their types in the response. Default is `true`.
version | Boolean | Whether to include the document version as a match.
include_named_queries_score | Boolean | Whether to return scores with named queries. Default is `false`.

### The `preference` query parameter

The `preference` query parameter specifies the shards or nodes on which OpenSearch should perform the search. The following are valid values:

- `_primary`: Perform the search only on primary shards.
- `_replica`: Perform the search only on replica shards.
- `_primary_first`: Perform the search on primary shards but fail over to other available shards if primary shards are not available.
- `_replica_first`: Perform the search on replica shards but fail over to other available shards if replica shards are not available.
- `_local`: If possible, perform the search on the local node's shards.
- `_prefer_nodes:<node-id-1>,<node-id-2>`: If possible, perform the search on the specified nodes. Use a comma-separated list to specify multiple nodes.
- `_shards:<shard-id-1>,<shard-id-2>`: Perform the search only on the specified shards. Use a comma-separated list to specify multiple shards. When combined with other preferences, the `_shards` preference must be listed first. For example, `_shards:1,2|_replica`.
- `_only_nodes:<node-id-1>,<node-id-2>`: Perform the search only on the specified nodes. Use a comma-separated list to specify multiple nodes.
- `<string>`: Specifies a custom string to use for the search. The string cannot start with an underscore character (`_`). Searches with the same custom string are routed to the same shards.

## Request body

All fields are optional.

Field | Type | Description
:--- | :--- | :---
aggs | Object | In the optional `aggs` parameter, you can define any number of aggregations. Each aggregation is defined by its name and one of the types of aggregations that OpenSearch supports. For more information, see [Aggregations]({{site.url}}{{site.baseurl}}/aggregations/).
docvalue_fields | Array of objects | The fields that OpenSearch should return using their docvalue forms. Specify a format to return results in a certain format, such as date and time.
fields | Array | The fields to search for in the request. Specify a format to return results in a certain format, such as date and time.
explain | String | Whether to return details about how OpenSearch computed the document's score. Default is `false`.
from | Integer | The starting index to search from. Default is 0.
indices_boost | Array of objects | Values used to boost the score of specified indexes. Specify in the format of &lt;index&gt; : &lt;boost-multiplier&gt;
min_score | Integer | Specify a score threshold to return only documents above the threshold.
query | Object | The [DSL query]({{site.url}}{{site.baseurl}}/opensearch/query-dsl/index/) to use in the request.
seq_no_primary_term | Boolean | Whether to return sequence number and primary term of the last operation of each document hit.
size | Integer | How many results to return. Default is 10.
_source | | Whether to include the `_source` field in the response.
stats | String | Value to associate with the request for additional logging.
terminate_after | Integer | The maximum number of matching documents (hits) OpenSearch should process before terminating the request. Default is 0.
timeout | Time | How long to wait for a response. Default is no timeout.
version | Boolean | Whether to include the document version in the response.

## Example request

```json
GET /movies/_search
{
  "query": {
    "match": {
      "text_entry": "I am the night"
    }
  }
}
```
{% include copy-curl.html %}


## Response body fields

```json
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.0,
    "hits": [
      {
        "_index": "superheroes",
        "_id": "1",
        "_score": 1.0,
        "_source": {
          "superheroes": [
            {
              "Hero name": "Superman",
              "Real identity": "Clark Kent",
              "Age": 28
            },
            {
              "Hero name": "Batman",
              "Real identity": "Bruce Wayne",
              "Age": 26
            },
            {
              "Hero name": "Flash",
              "Real identity": "Barry Allen",
              "Age": 28
            },
            {
              "Hero name": "Robin",
              "Real identity": "Dick Grayson",
              "Age": 15
            }
          ]
        }
      }
    ]
  }
}
```

## The `ext` object

Starting with OpenSearch 2.10, plugin authors can add an `ext` object to the search response. The `ext` object contains plugin-specific response fields. For example, in conversational search, the result of retrieval-augmented generation (RAG) is a single "hit" (answer). Plugin authors can include this answer in the search response as part of the `ext` object so that it is separate from the search hits. In the following example response, the RAG result is in the `ext.retrieval_augmented_generation.answer` field:

```json
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 3,
    "successful": 3,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 110,
      "relation": "eq"
    },
    "max_score": 0.55129033,
    "hits": [
      {
       "_index": "...",
        "_id": "...",
        "_score": 0.55129033,
        "_source": {
          "text": "...",
          "title": "..."
        }
      },
      {
      ...
      }
      ...
      {
      ...
      }
    ],
  }, // end of hits
  "ext": {
    "retrieval_augmented_generation": { // a search response processor
      "answer": "RAG answer"
    }
  }
} 
```
