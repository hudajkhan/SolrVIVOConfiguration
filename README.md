# SolrVIVOConfiguration
Testing Solr 7.4 configuration with VIVO 

Solr 7 is a self-contained Solr.  Experimenting with setting up a Solr instance to work with VIVO.
The directory above is under the installed Solr directory at this location:
server/solr

Solrconfig was copied from Solr's own solrconfig (I can review what updates were made but it is mostly the same). 

The schema file was copied from VIVO's original schema with some updates as required to allow Solr to function (and account for changes):
- solrQueryParser defaultOperator="AND" : no longer supported so commented out
- <fieldType name="text_unstemmed" class="solr.TextField" positionIncrementGap="100">, <fieldType name="textgen" class="solr.TextField" positionIncrementGap="100">,<fieldType name="lowercase" class="solr.TextField" positionIncrementGap="100">, <fieldtype name="edgengram_stemmed"  class="solr.TextField">: filter class="solr.StopFilterFactory" does not need the enablePositionIncrements="true" flag 
- <fieldType name="location_rpt"..> uses distanceUnits instead of units
- <similarity class="org.apache.lucene.search.similarities.DefaultSimilarity"> is not recognized and was commented out. (I need to check but I think a class needs to be specified if the default behavior is not desired.)
- side="front" removed from  <filter class="solr.EdgeNGramFilterFactory">
 
 
On the VIVO end, runtime.properties (in this example) uses vitro.local.solr.url = http://localhost:8984/solr/collection1.

Originally, in order to get the Solr smoke tests to work, I had to use http://localhost:8984/solr/#/collection1 (i.e. an extra /#/ between solr and the name of the Solr collection). Note the difference between this and the earlier syntax where the collection name did not have to be included and the # was not required.  Without the #, the Solr Smoke tests fail because the tryToConnect method's HttpGet does not return a valid status code.  On the other hand, both http://localhost:8984/solr/#/collection1/admin/ping and  http://localhost:8984/solr/collection1 return valid results for the pinging code in SolrPinger, with the first returning an HTML response and the second returning JSON. Most importantly, if you keep the Solr url as http://localhost:8984/solr/collection1, the search index appears to be indexing new content (it shows a new test faculty member I set up).  I tested this out by turning off the Solr smoke tests in startup listeners and then using the http://localhost:8984/solr/collection1 URL.  