Proposed SPARQL 1.1 Protocol Tests
==================================

This repository contains code which attempts to validate implementations of the
[SPARQL 1.1 Protocol](http://www.w3.org/TR/sparql11-protocol/). It is a work in
progress, and subject to change as the Protocol spec continues towards becoming
a W3C recommendation, and as we
(the [SPARQL Working Group](http://www.w3.org/2009/sparql/)) continue to
evaluate existing existing implementations.

The validator is implemented as a perl CGI, and depends on packages available
from [CPAN](https://metacpan.org/):

* [JSON](https://metacpan.org/release/JSON)
* [LWP::UserAgent](https://metacpan.org/release/libwww-perl)
* [RDF::Trine](https://metacpan.org/release/RDF-Trine)
* [URI::Escape](https://metacpan.org/release/URI)

Setup
-----

### Configuration

When accessed without any query parameters, the CGI provides an HTML form that
may be used to validate a Protocol implementation. The parameters are:

* "query_url" -- query endpoint URL
* "update_url" -- update endpoint URL
* "software" -- The Protocol implementation IRI that will be used if [conneg](http://www.w3.org/Protocols/rfc2616/rfc2616-sec12.html) is used and requests RDF

The following parameters *should* also be accounted for (in a future version):

* does the default graph change based on other graphs (e.g. acts as the union of named graphs)?

### Requirements

It is assumed that the Protocol implementation provides support for all of SPARQL (1.0)
 and also SPARQL 1.1 Query/Update support for:

* Select expressions
* CLEAR ALL
* CLEAR DEFAULT
* CLEAR NAMED

The graph store available to query and update dataset operations must contain 
the triples available at the following URLs (in respective named graphs):

* http://@@.example.org/data0.rdf
* http://@@.example.org/data1.rdf
* http://@@.example.org/data2.rdf
* http://@@.example.org/data3.rdf

Finally, it is assumed that implementations can produce application/rdf+xml and
application/sparql-results+xml when requested using [conneg](http://www.w3.org/Protocols/rfc2616/rfc2616-sec12.html).


Tests
-----

The following tests are implemented (or are planned for implementation):

### Negative tests

Negative tests are expected to fail with either 4xx or 5xx response codes.

* bad-query-method - invoke query operation with a method other than GET or POST

		PUT /sparql?query=ASK%20%7B%7D

* bad-multiple-queries - invoke query operation with more than one query string

		GET /sparql?query=ASK%20%7B%7D&query=SELECT%20%2A%20%7B%7D

* bad-query-wrong-media-type - invoke query operation with a POST with media type that's not url-encoded or application/sparql-query

		POST /sparql/ HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: text/plain
		Content-Length: XXX
		
		ASK {}

* bad-query-missing-form-type - invoke query operation with url-encoded body, but without application/x-www-url-form-urlencoded media type

		POST /sparql/ HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Length: XXX
		
		query=ASK%20%7B%7D

* bad-query-missing-direct-type - invoke query operation with SPARQL body, but without application/sparql-query media type

		POST /sparql/ HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Length: XXX
		
		ASK {}

* bad-query-non-utf8 - invoke query operation with direct POST, but with a non-UTF8 encoding (UTF-16)

		### (content body encoded in utf-16)
		POST /sparql/ HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/sparql-query; charset=UTF-16
		Content-Length: XXX
		
		ASK {}

* bad-query-syntax - invoke query operation with invalid query syntax (4XX result)

		GET /sparql?query=ASK%20%7B

* bad-update-get - invoke update operation with GET

		GET /sparql?update=CLEAR%20ALL

* bad-multiple-updates - invoke update operation with more than one update string

		POST /sparql/ HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/x-www-url-form-urlencoded
		Content-Length: XXX
		
		update=CLEAR%20NAMED&update=CLEAR%20DEFAULT

* bad-update-wrong-media-type - invoke update operation with a POST with media type that's not url-encoded or application/sparql-update

		POST /sparql/ HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: text/plain
		Content-Length: XXX
		
		CLEAR NAMED

* bad-update-missing-form-type - invoke update operation with url-encoded body, but without application/x-www-url-form-urlencoded media type

		POST /sparql/ HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Length: XXX
		
		update=CLEAR%20NAMED

* bad-update-missing-direct-type - invoke update operation with SPARQL body, but without application/sparql-update media type

		POST /sparql/ HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Length: XXX
		
		CLEAR NAMED

* bad-update-non-utf8 - invoke update operation with direct POST, but with a non-UTF8 encoding

		### (content body encoded in utf-16)
		POST /sparql/ HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/sparql-update; charset=UTF-16
		Content-Length: XXX
		
		CLEAR NAMED

* bad-update-syntax - invoke update operation with invalid update syntax (4XX result)

		POST /sparql/ HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/x-www-url-form-urlencoded
		Content-Length: XXX
		
		update=CLEAR%20XYZ

* bad-update-dataset-conflict - invoke update with both using-graph-uri/using-named-graph-uri parameter and USING/WITH clause

		POST /sparql/ HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/x-www-url-form-urlencoded
		Content-Length: XXX
		
		using-named-graph-uri=http%3A%2F%2Fexample%2Fpeople&update=%09%09PREFIX%20foaf%3A%20%20%3Chttp%3A%2F%2Fxmlns.com%2Ffoaf%2F0.1%2F%3E%0A%09%09WITH%20%3Chttp%3A%2F%2Fexample%2Faddresses%3E%0A%09%09DELETE%20%7B%20%3Fperson%20foaf%3AgivenName%20%27Bill%27%20%7D%0A%09%09INSERT%20%7B%20%3Fperson%20foaf%3AgivenName%20%27William%27%20%7D%0A%09%09WHERE%20%7B%0A%09%09%09%3Fperson%20foaf%3AgivenName%20%27Bill%27%0A%09%09%7D%0A


### Positive tests

Positive tests are expected to succeed with either 2xx or 3xx response codes.
Some of the following tests also test the response content for expected results.

* query-get - query via GET

		GET /sparql?query=ASK%20%7B%7D

* query-post-form - query via URL-encoded POST

		POST /sparql/ HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/x-www-url-form-urlencoded
		Content-Length: XXX
		
		query=ASK%20%7B%7D

* query-post-direct - query via POST directly

		POST /sparql/ HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/sparql-query
		Content-Length: XXX
		
		ASK {}

* query-dataset-default-graph - query with protocol-specified default graph

		POST /sparql/?default-graph-uri=http%3A%2F%2F@@.example.org%2Fdata1.rdf HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/sparql-query
		Content-Length: XXX
		
		ASK { <http://@@.example.org/data1.rdf> ?p ?o }

* query-dataset-default-graphs - query with protocol-specified default graphs

		POST /sparql/?default-graph-uri=http%3A%2F%2F@@.example.org%2Fdata1.rdf&default-graph-uri=http%3A%2F%2F@@.example.org%2Fdata2.rdf HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/sparql-query
		Content-Length: XXX
		
		ASK { <http://@@.example.org/data1.rdf> ?p ?o . <http://@@.example.org/data2.rdf> ?p ?o }

* query-dataset-named-graphs - query with protocol-specified named graphs

		POST /sparql/?named-graph-uri=http%3A%2F%2F@@.example.org%2Fdata1.rdf&named-graph-uri=http%3A%2F%2F@@.example.org%2Fdata2.rdf HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/sparql-query
		Content-Length: XXX
		
		ASK { GRAPH ?g { ?s ?p ?o } }

* query-dataset-full - query with protocol-specified dataset (both named and default graphs)

		POST /sparql/?default-graph-uri=http%3A%2F%2F@@.example.org%2Fdata3.rdf&named-graph-uri=http%3A%2F%2F@@.example.org%2Fdata1.rdf&named-graph-uri=http%3A%2F%2F@@.example.org%2Fdata2.rdf HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/sparql-query
		Content-Length: XXX
		
		SELECT ?g ?x ?s { ?x ?y ?o  GRAPH ?g { ?s ?p ?o } }

* query-multiple-dataset - query specifying dataset in both query string and protocol; test for use of protocol-specified dataset (test relies on the endpoint allowing client-specified RDF datasets; returns 400 otherwise)

		POST /sparql/?default-graph-uri=http%3A%2F%2F@@.example.org%2Fdata2.rdf HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/sparql-query
		Content-Length: XXX
		
		ASK FROM <http://@@.example.org/data1.rdf> { <data1.rdf> ?p ?o }

* query-content-type-select - query appropriate content type (expect one of: XML, JSON, CSV, TSV)

		POST /sparql/ HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/sparql-query
		Content-Length: XXX
		
		SELECT (1 AS ?value) {}

* query-content-type-ask - query appropriate content type (expect one of: XML, JSON)

		POST /sparql/ HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/sparql-query
		Content-Length: XXX
		
		ASK {}

* query-content-type-describe - query appropriate content type (expect one of: RDF/XML, Turtle, N-Triples, RDFa)

		POST /sparql/ HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/sparql-query
		Content-Length: XXX
		
		DESCRIBE <http://example.org/>

* query-content-type-construct - query appropriate content type (expect one of: RDF/XML, Turtle, N-Triples, RDFa)

		POST /sparql/ HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/sparql-query
		Content-Length: XXX
		
		CONSTRUCT { <s> <p> 1 } WHERE {}

* update-dataset-default-graph - update with protocol-specified default graph

		POST /sparql?using-graph-uri=http%3A%2F%2Fkasei.us%2F2009%2F09%2Fsparql%2Fdata%2Fdata1.rdf HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/sparql-update
		Content-Length: XXX
		
		PREFIX dc: <http://purl.org/dc/terms/>
		PREFIX foaf: <http://xmlns.com/foaf/0.1/>
		CLEAR ALL ;
		INSERT DATA { GRAPH <http://kasei.us/2009/09/sparql/data/data1.rdf> { <http://kasei.us/2009/09/sparql/data/data1.rdf> a foaf:Document } } ;
		INSERT {
			GRAPH <http://example.org/protocol-update-dataset-test/> {
				?s a dc:BibliographicResource
			}
		}
		WHERE {
			?s a foaf:Document
		}


		POST /sparql HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/sparql-query
		Content-Length: XXX
		
		ASK {
			GRAPH <http://example.org/protocol-update-dataset-test/> {
				<http://kasei.us/2009/09/sparql/data/data1.rdf> a <http://purl.org/dc/terms/BibliographicResource>
			}
		}

* update-dataset-default-graphs - update with protocol-specified default graphs

		POST /sparql?using-graph-uri=http%3A%2F%2Fkasei.us%2F2009%2F09%2Fsparql%2Fdata%2Fdata1.rdf&using-graph-uri=http%3A%2F%2Fkasei.us%2F2009%2F09%2Fsparql%2Fdata%2Fdata2.rdf HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/sparql-update
		Content-Length: XXX
		
		PREFIX dc: <http://purl.org/dc/terms/>
		PREFIX foaf: <http://xmlns.com/foaf/0.1/>
		CLEAR ALL ;
		INSERT DATA { GRAPH <http://kasei.us/2009/09/sparql/data/data1.rdf> { <http://kasei.us/2009/09/sparql/data/data1.rdf> a foaf:Document } } ;
		INSERT {
			GRAPH <http://example.org/protocol-update-dataset-graphs-test/> {
				?s a dc:BibliographicResource
			}
		}
		WHERE {
			?s a foaf:Document
		}


		POST /sparql HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/sparql-query
		Content-Length: XXX
		
		ASK {
			GRAPH <http://example.org/protocol-update-dataset-test/> {
				<http://kasei.us/2009/09/sparql/data/data1.rdf> a <http://purl.org/dc/terms/BibliographicResource> .
				<http://kasei.us/2009/09/sparql/data/data2.rdf> a <http://purl.org/dc/terms/BibliographicResource> .
			}
		}

* update-dataset-named-graphs - update with protocol-specified named graphs

		@@ fill in details

* update-dataset-full - update with protocol-specified dataset (both named and default graphs)

		@@ fill in details

* update-post-form - update via URL-encoded POST

		POST /sparql/ HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/x-www-url-form-urlencoded
		Content-Length: XXX
		
		update=CLEAR%20ALL

* update-post-direct - update via POST directly

		POST /sparql/ HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/sparql-update
		Content-Length: XXX
		
		CLEAR ALL

* update-base-uri - test for service-defined BASE URI ("which MAY be the service endpoint")

*NOTE*: For this to test that an *update* has a default BASE URI, the test will have to span multiple requests: an insert that assumes a base uri, followed by a query to extract the new value and test it for base uri resolution.

		POST /sparql/ HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/sparql-update
		Content-Length: XXX
		
		CLEAR GRAPH <http://example.org/protocol-base-test/> ; INSERT DATA { GRAPH <http://example.org/protocol-base-test/> { <http://example.org/s> <http://example.org/p> <test> } }


		POST /sparql/ HTTP/1.1
		Host: www.example
		User-agent: sparql-client/0.1
		Content-Type: application/sparql-query
		Content-Length: XXX
		
		SELECT ?o WHERE { GRAPH <http://example.org/protocol-base-test/> { <http://example.org/s> <http://example.org/p> ?o } }
