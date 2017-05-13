---
layout: post
title:  "How To Make Elasticsearch Fast"
date:   2017-05-12 12:11:28
categories: elasticsearch
---

Add the following to the settings of an index you want to speed up:

	{
	  "settings": {
	    "index": {
	      "queries": {
	        "cache": {
	          "everything": true
	        }
	      }
	    }
	  }
	}

Why?

    // for test purposes only
    public static final Setting<Boolean> INDEX_QUERY_CACHE_EVERYTHING_SETTING =
        Setting.boolSetting("index.queries.cache.everything", false, Property.IndexScope);

    // This setting is an escape hatch in case not caching term queries would slow some users down
    // Do not document.
    public static final Setting<Boolean> INDEX_QUERY_CACHE_TERM_QUERIES_SETTING =
        Setting.boolSetting("index.queries.cache.term_queries", false, Property.IndexScope);

Alternatively, if you're serious about searching, use [Solr](http://lucene.apache.org/solr/). Bonus points if you [back it with HDFS](https://cwiki.apache.org/confluence/display/solr/Running+Solr+on+HDFS) so you never have to sync data.

Good luck!