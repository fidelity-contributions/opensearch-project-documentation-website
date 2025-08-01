# API Style Guide

This guide provides the basic structure for creating OpenSearch API documentation. It includes the various elements that we feel are most important to creating complete and useful API documentation, as well as description and examples where appropriate.

Depending on the intended purpose of the API, *some sections will be required while others may not be applicable*.

Use the [API_TEMPLATE](templates/API_TEMPLATE.md) to create an API documentation page.

### A note on terminology

Terminology for API parameters varies in the software industry, where two or even three names may be used to label the same type of parameter. For consistency, we use the following nomenclature for parameters in our API documentation:
* *Path parameter* – "path parameter" and "URL parameter" are sometimes used synonymously. To avoid confusion, we use "path parameter" in this documentation.
* *Query parameter* – This parameter name is often used synonymously with "request parameter." We use "query parameter" to be consistent.

### General usage for code elements

When you describe any code element in a sentence, such as an API, a parameter, or a field, you can use the noun name. 
  *Example usage*:
  The time field provides a timestamp for job completion.

When you provide an exact example with a value, you can use the code element in code font. 
  *Example usage*: 
  The response provides a value for `time_field`, such as “timestamp.” 

Provide a REST API call example in `json` format. Optionally, also include the `curl` command if the call can only be executed in a command line.

## Basic elements for documentation

The following sections describe the basic API documentation structure. Each section is discussed under its respective heading. Include only those elements appropriate to the API. 

Depending on where the documentation appears within a section or subsection, heading levels may be adjusted to fit with other content.

1. Name of API 
1. Endpoints 
1. (Optional) Path parameters 
1. (Optional) Query parameters 
1. (Optional) Request body fields 
1. Example request 
1. Example response 
1. (Optional) Response body fields 

## API name

Provide an API name that describes its function, followed by a description of its top use case and any usage recommendations.

*Example function*: "Autocomplete queries"

Use sentence capitalization for the heading (for example, "Create or update mappings"). When you refer to the API operation, you can use lowercase with code font.

If there is a corresponding OpenSearch Dashboards feature, provide a “See also” link that references it. 
*Example*: “To learn more about monitor findings, see [Document findings](https://docs.opensearch.org/latest/monitoring-plugins/alerting/monitors/#document-findings)."

If applicable, provide any caveats to its usage with a note or tip, as in the following example:

"If you use the Security plugin, make sure you have the appropriate permissions."
(To set this point in note-style format, follow the text on the next line with {: .note})

### Endpoints

For relatively complex API calls that include path parameters, it's sometimes a good idea to provide an example so that users can visualize how the request is properly formed. This section is optional and includes examples that illustrate how the endpoint and path parameters fit together in the request. The following is an example of this section for the nodes stats API:

````
```json
GET /_nodes/stats
GET /_nodes/<node_id>/stats
GET /_nodes/stats/<metric>
GET /_nodes/<node_id>/stats/<metric>
GET /_nodes/stats/<metric>/<index_metric>
GET /_nodes/<node_id>/stats/<metric>/<index_metric>
```
````

### Path parameters

While the API endpoint states a point of entry to a resource, the path parameter acts on the resource that precedes it. Path parameters come after the resource name in the URL.

In the following example, the resource is `scroll` and its path parameter is `<scroll_id>`:

```json
GET _search/scroll/<scroll_id>
```

Introduce what the path parameters can do at a high level. Provide a table with parameter names and descriptions. Include a table with the following columns:
*Parameter* – Parameter name in plain font.
*Data type* – Data type capitalized (such as Boolean, String, or Integer).
*Description* – Sentence to describe the parameter function, default values or range of values, and any usage examples.

```md
Parameter | Data type | Description
:--- | :--- | :---
```

### Query parameters

In terms of placement, query parameters are always appended to the end of the URL and located to the right of the operator "?". Query parameters serve the purpose of modifying information to be retrieved from the resource.

In the following example, the endpoint is `aliases` and its query parameter is `v` (provides verbose output):

```json
GET _cat/aliases?v
```

Include a paragraph that describes how to use the query parameters with an example in code font. Include the query parameter operator "?" to delineate query parameters from path parameters.

For GET and DELETE APIs: Introduce what you can do with the optional parameters. Include a table with the same columns as the path parameter table.

```md
Parameter | Data type | Description
:--- | :--- | :---
```

### Request body fields

For PUT and POST APIs: Introduce what the request fields are allowed to provide in the body of the request.

Include a table with these columns: 
*Field* – Field name in plain font.
*Data type* – Data type capitalized (such as Boolean, String, or Integer).
*Description* – Sentence to describe the field’s function, default values or range of values, and any usage examples.

```md
Field | Data type | Description
:--- | :--- | :--- 
```

## Example request

Make sure that you test the request. You can optionally add a descriptive sentence for the request. See the following examples.

````md
## Example request

```json
GET /sample-index1/_settings
```
````

````md
## Example request

The following request copies all of your field mappings and settings from a source index to a destination index:

```json
POST _reindex
{
   "source":{
      "index":"sample-index-1"
   },
   "dest":{
      "index":"sample-index-2"
   }
}
```
````

To showcase multiple example requests, include a request description in the heading:

````markdown
## Example request: Searching for all agents

```json
POST /_plugins/_ml/agents/_search
{
  "query": {
    "match_all": {}
  },
  "size": 1000
}
```
{% include copy-curl.html %}

## Example request: Searching for agents of a certain type

```json
POST /_plugins/_ml/agents/_search
{
  "query": {
    "term": {
      "type": {
        "value": "flow"
      }
    }
  }
}
```
{% include copy-curl.html %}
````

If the responses are significantly different, you can include a response for each request. If the responses are the same, include one response at the end of all requests.

## Example response

Include a JSON example response to show what the API returns. You can optionally add a descriptive sentence for the response. See the following examples.

````md
## Example response

```json
{
  "sample-index1": {
    "settings": {
      "index": {
        "creation_date": "1622672553417",
        "number_of_shards": "1",
        "number_of_replicas": "1",
        "uuid": "GMEA0_TkSaamrnJSzNLzwg",
        "version": {
          "created": "135217827",
          "upgraded": "135238227"
        },
        "provided_name": "sample-index1"
      }
    }
  }
}
```
````

````md
## Example response: Get all stats

The response contains statistics for each node:

```json
{
  "zbduvgCCSOeu6cfbQhTpnQ" : {
    "ml_executing_task_count" : 0
  },
  "54xOe0w8Qjyze00UuLDfdA" : {
    "ml_executing_task_count" : 0
  }
}
````

Enclose long responses in a `details` block:

````md
<details open markdown="block">
  <summary>
    Response
  </summary>
  {: .text-delta}

```json
{
  "took" : 4,
  "timed_out" : false,
  "total" : 0,
  "updated" : 0,
  "created" : 0,
  "deleted" : 0,
  "batches" : 0,
  "version_conflicts" : 0,
  "noops" : 0,
  "retries" : {
    "bulk" : 0,
    "search" : 0
  },
  "throttled_millis" : 0,
  "requests_per_second" : -1.0,
  "throttled_until_millis" : 0,
  "failures" : [ ]
}
```
</details>
````

### Response body fields

For PUT and POST APIs: Define all allowable response fields that can be returned in the body of the response.

```md
Field | Data type | Description
:--- | :--- | :---
```