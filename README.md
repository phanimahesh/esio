#Esio: Elasticsearch I/O

[![Build Status](https://secure.travis-ci.org/zalando/esio.svg?branch=master)](http://travis-ci.org/zalando/esio)

Esio is an Erlang library that provides an HTTP client for [Elasticsearch](https://www.elastic.co/products/elasticsearch), which offers a sophisticated [RESTful API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs.html). It implements a data access layer for Erlang applications and makes data and objects semantic-friendly with Erlang. 

Unlike other, similar projects, Esio does not aim to provide a generic HTTP-proxy approach, but instead: 
* uses an strict identity schema to resolve naming crises and the hierarchical nature of index/type/key notation
* offers query templates to counter the complexity of DSL 
* uses Actor models for data streaming
* provides Erlang-friendly pattern matching  

Esio is still under development. Supplementary files:
* [design considerations](doc/design.md)
* [usage examples](doc/example.md) 

### Installing/Using Esio

The latest version of Esio is available from the master branch. All development, including new features and bug fixes, take place on the master branch using forking and pull requests (as described in the [contribution guidelines](doc/contribution.md)). 

To use and develop Esio, you need:
* Erlang/OTP 18.x or later
* Elasticsearch 2.x or later

This library uses [rebar3](http://www.rebar3.org). Use the following code snippet to include Esio in your `rebar.config`:
```
   {esio, ".*",
      {git, "https://github.com/zalando/esio", {branch, master}}
   }
``` 

### Running Esio
Esio exposes all its functionality through its [public interface](src/esio.erl). This enables client applications to manage socket connections to Elasticsearch, execute basic key/value (hashmap-like) operations and search arbitrary documents in Elasticsearch. 

You can experiment with Esio's features in your development console. This requires downloading Erlang/OTP version 18.0 or later and Elasticsearch. Elasticsearch's [Docker container](https://hub.docker.com/_/elasticsearch/) is easiest way to run a standalone instance of Elasticsearch for development purposes.

```
docker run -it -p 9200:9200 elasticsearch
```

Build and run Esio in your development console:     
```
make && make run
```

Let's create an index and store some documents:

```erlang
%% 
%% start library
esio:start().

%%
%% open a socket to Elastic Search
{ok, Schema} = esio:socket("http://127.0.0.1:9200").

%%
%% create a new index with name `z`
esio:put(Schema, "z", #{settings => #{number_of_shards => 3, number_of_replicas => 1}}).

%%
%% open a socket bounded to index `z` and type `default` 
{ok, Sock} = esio:socket("http://127.0.0.1:9200/z/default").

%%
%% put documents to `default` mapping of the index `z`
esio:put(Sock, "1", #{title => <<"red color">>, tags => [red]}).
esio:put(Sock, "2", #{title => <<"yellow color">>, tags => [yellow]}).
esio:put(Sock, "3", #{title => <<"green color">>, tags => [green]}).

%%
%% put documents to `other` mapping of the index `z`
esio:put(Sock, {urn, other, "4"}, #{title => <<"black color">>, tags => [black]}).


%%
%% get document from `default` mapping of the index `z` identifiable by key `1`
esio:get(Sock, "1").

%%
%% get document for `other` mapping of the index `z`
esio:get(Sock, {urn, other, "4"}).


%%
%% match a documents to a pattern  
esio:match(Sock, #{tags => yellow}).

%%
%% close connection
esio:close(Sock).
```

See other examples [here](doc/example.md).


### Contributing & Fixing Bugs
See our [contribution guidelines](doc/contribution.md) for details on how to submit PRs. For bugs, see our [bug reporting](doc/bugs.md) guidelines and let us know about new bugs via the [Issues Tracker](https://github.com/zalando/esio/issues). 

### Contact

* email: dmitry.kolesnikov@zalando.fi

### Changelog

The library uses [semantic versions](http://semver.org) to identify stable releases. 
 
* [1.0.0](https://github.com/zalando/esio) - improved identity schema and Elastic Search 5.x integration, not compatible with version 0.1.x
* [0.1.3](https://github.com/zalando/esio/releases/tag/0.1.3) - stable elastic search i/o
  

### License

The MIT License (MIT)

Copyright (c) 2016 Zalando SE

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
