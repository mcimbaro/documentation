---
title: Importing Data From an External Elasticsearch® Database
nav: Importing Data
modified_at: 2024-08-23 00:00:00
tags: databases elasticsearch dump restore migration
index: 3
---

This tutorial aims at transferring all the data from a remote Elasticsearch® database
(from another provider) to an Elasticsearch® instance provisioned through the [Scalingo
For Elasticsearch® Addon]({% post_url databases/elasticsearch/2000-01-01-start %}).

{% include info_command_line_tool.md %}

## Requirements

The [elasticsearch-dump](https://github.com/elasticsearch-dump/elasticsearch-dump) tool should be available on your system, etheir via `npm` or `docker`.

The source Elasticsearch® instance should be backuped to a file (using elasticsearch-dump) or be accessible on the Internet.

## Install elasticsearch-dump

[elasticsearch-dump](https://github.com/elasticsearch-dump/elasticsearch-dump) is a tool allowing you to dump indexes from your Elasticsearch® or OpenSearch® database.

You can use the elasticsearch-dump tool by etheir one of the two following way:
* System installation via `npm`. Install the dependency at a project or system level:
  * At project level: `npm install elasticdump`
  * At system level (may require super-user rights): `npm install elasticdump -g`
* By using `docker`, pull the image by using the following command: `docker pull elasticdump/elasticsearch-dump`

## Data stored on your cluster

Your Elasticsearch® cluster store a wide range of objects:
* `index`: [An index](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html) is the fundamental object allowing search
* `settings` : [An index level settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules.html#index-modules-settings)
* `analyzer`: [An analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analyzer.html) is the object responsible for handling [text analysis](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html) on your cluster
* `data` : Your raw data stored on your cluster
* `mapping` : [A mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html) is the object reponsible for deciding how a document, and the fields it contains, are stored and indexes
* `policy`
* `alias` : [An alias](https://www.elastic.co/guide/en/elasticsearch/reference/current/aliases.html) is a second name for a group of data streams or indices
* `template`
* `component_template` : [A component template](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-component-template.html) is a building block to declare an index template that specify index mapping, settings and aliases.
* `index_template` : [An index template](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-templates.html) is a set of configuration to apply to an index upon creation

Depending on your use case, you may need to export some or almost all of the types below.

For a decent cluster export, you may want to export the following types: `index`, `analyzer` and `data`.

## Dump your database to a file

With a system installation:
```bash
elasticdump \
  --input=http://my-elasticsearch.tld/my_index \
  --output=~/index_analyser.json \
  --type=analyzer
```

With Docker:
```bash
docker run --rm -ti -v ~/:/data elasticdump/elasticsearch-dump \
  --input=http://my-elasticsearch.tld/my_index \
  --output=/data/index_analyser.json \
  --type=analyzer
```

If you cluster is protected by Basic-Auth credentials, you can specify them before the domain name, eg: `http://<user>:<password>@my-elasticsearch.tld/my_index`.

Repeat the step below with all types you want to export.

## Restore a dump from a file

With a system installation:
```bash
elasticdump \
  --input=~/index_analyser.json \
  --output=http://my-elasticsearch.tld/my_index \
  --type=analyzer
```

With Docker:
```bash
docker run --rm -ti -v ~/:/data elasticdump/elasticsearch-dump \
  --input=/data/index_analyser.json \
  --output=http://my-elasticsearch.tld/my_index \
  --type=analyzer
```

Repeat the step below with all types previously exported.

## Dump and restore at once

For the direct transfer to work:
* Your source database should be accessible on the Internet
* A DB-tunnel should be opened with your Scalingo database
  * `scalingo -a my-app db-tunnel SCALINGO_ELASTICSEARCH_URL`. This command will expose your database to port `10000` by default.

With a system installation:
```bash
elasticdump \
  --input=http://my-elasticsearch.tld/my_index \
  --output=http://127.0.0.1:10000/my_index \
  --type=analyzer
```

With Docker:
```bash
docker run --rm -ti \
  --input=http://my-elasticsearch.tld/my_index \
  --output=http://127.0.0.1:10000/my_index \
  --type=analyzer
```

Repeat the step below with all types you want to transfer.
