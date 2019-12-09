---
layout: post
title:  "ElasticSearch Percolator Use Case for Document Classification"
excerpt: ""
date:   2019-12-05 12:00:00 +0100
categories: python
tags: python elasticsearch percolator
---

Currently at Allo-Media, we use Elasticsearch in its general workflow which is to create an index and store documents holding our phone call audio transcripts metadata, and then allowing to search through these documents given some business criteria like: "Give me all phone calls from client Acme, where the customer speaks about the French strike".

The percolator feature from Elasticsearch allows to make a reverse search. We store search queries as documents in its own index, and then we can percolate new call documents and retrieve what search queries match. One use case to use the percolator is document classification.

For example, say that we want to tag with `Check sent` all documents mentioning that the user has already sent a bank check. We would have the following search query: 
```
("I've sent" | "I've already sent") ("check")
```

So first, we need to create an index to store the search queries with the following mapping: 
```
PUT /search-perco
{
  "mappings": {
    "_doc": {
      "properties": {
        "tag_uuid": {
          "type": "text"
        },
        "tag_name": {
          "type": "text"
        },
        "content": {
          "type": "text"
        },
        "query": {
          "type": "percolator"
        }
      }
    }
  }
}
```

- The `tag_*` fields are used for document classification
- The `query` field of type `percolator` is used to index the search query documents, storing a [query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html) in JSON
- The `content` field is used to preprocess the percolating documents.

Once the index is created, we can now store our search query documents, like the following one: 
```
PUT /search-perco/_doc/1?refresh
{
  "query": {
      "bool": {
          "must": [
              {
                  "simple_query_string": {
                      "query": "("I've sent" | "I've already sent") ("check")",
                      "fields": [
                          "content"
                      ],
                      "default_operator": "and"
                  }
              }
          ]
      }
  },
  "tag_uuid": "2f86ad85-4c09-4ef3-bb6e-100d129018e9",
  "tag_name": "Check sent",
}
```

And if we search through this index, we will retrieve our newly added search query document: 
```
GET search-perco/_search
{
  "query": {"match_all": {}}
}

{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 1,
    "hits": [
      {
        "_index": "search-perco",
        "_type": "_doc",
        "_id": "1",
        "_score": 1,
        "_source": {
          "query": ...
          "tag_uuid": "2f86ad85-4c09-4ef3-bb6e-100d129018e9",
          "tag_name": "Check sent",
        }
      }
    ]
  }
}
```

Now it's time to percolate call documents via the percolate query: 
```
GET /search-perco/_search
{
  "_source": [
    "tag_uuid",
    "tag_name"
  ],
  "query": {
    "percolate": {
      "field": "query",
      "documents": [{`
        "unique_id": "2f86ad85-4c09-4ef3-bb6e-100d129018e7",
        "timestamp": "2018-01-02T18:13:30+00:00",
        "duration": 322,
        "transcribed": true,
        "client_name": "Acme",
        "content": "I've already sent to you a bank check last week..."
      }]
    }
  },
  "highlight": {
    "fields": {
      "content": {}
    }
  }
}
```

Elasticsearch providing the following response: 
```
{
  "took": 37,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.8630463,
    "hits": [
      {
        "_index": "search-perco",
        "_type": "_doc",
        "_id": "1",
        "_score": 0.8630463,
        "_source": {
          "tag_name": "Check sent",
          "tag_uuid": "2f86ad85-4c09-4ef3-bb6e-100d129018e9"
        },
        "fields": {
          "_percolator_document_slot": [
            0
          ]
        },
        "highlight": {
          "content": [
            "<em>I've already sent</em> to you a bank <em>check</em> last week..."
          ]
        }
      }
    ]
  }
}
```

So here we see that our call document matched the search query tagged `Check sent`. We can use the highlighter to highlight the terms that have matched from the search query documents. The field `_percolator_document_slot` is useful when we send several documents to the `documents` field of the percolate query. And `max_score` and `_score` gives you the relevance score of matched documents. You can disable the score computing when using the percolate query using a [filter context](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html#filter-context).

We can also percolate existing documents by providing the index where they are stored, and their ids:
```
GET /search-perco/_search
{
    "query" : {
        "percolate" : {
            "field": "query",
            "index" : "call-index",
            "id" : "2"
        }
    }
}
```

You should care about optimizing text analysis during percolate time as suggested by the docs [Percolator optimization](https://www.elastic.co/guide/en/elasticsearch/reference/current/percolator.html#_optimizing_query_time_text_analysis).

Elasticsearch documentation:

- [Percolate query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-percolate-query.html)
- [ElasticSearch Percolator](https://www.elastic.co/guide/en/elasticsearch/reference/current/percolator.html)
