# SolrVIVOConfiguration
Testing Solr 7.4 configuration with VIVO 

Solr 7 is a self-contained Solr.  Experimenting with setting up a Solr instance to work with VIVO.
The contents above are from the collection directory under the installed Solr directory at this location:
server/solr/collection1 (where collection1 is the name of the Solr collection in this example)

The core.properties file under this directory defines where to find the schema and solrconfig. 

Solrconfig was copied from Solr's own solrconfig with some changes copying over the search handling from the original VIVO (Solr 4-compatible) schema, specifically the search handler (see < requestHandler name="/select" class="solr.SearchHandler" > ).  As mentioned below, the q.op parameter is no longer supported in schema.xml but is now in solrconfig for SearchHandler.  This is set to "AND", as it was originally, which means it was saying that all query terms had to be matched for results to return.  I have comments in the solrconfig saying I had to comment out lst name="defaults". I will need to revisit why/what effects this might have.

The schema file was copied from VIVO's original schema with some updates as required to allow Solr to function (and account for changes):
- solrQueryParser defaultOperator="AND" : no longer supported so commented out and the related behavior is encoded in solrconfig instead.
- < fieldType name="text_unstemmed" class="solr.TextField" positionIncrementGap="100" >, < fieldType name="textgen" class="solr.TextField" positionIncrementGap="100" >,< fieldType name="lowercase" class="solr.TextField" positionIncrementGap="100" >, < fieldtype name="edgengram_stemmed"  class="solr.TextField" >: filter class="solr.StopFilterFactory" does not need the enablePositionIncrements="true" flag 
- <fieldType name="location_rpt"..> uses distanceUnits instead of units
- <similarity class="org.apache.lucene.search.similarities.DefaultSimilarity"> is not recognized and was commented out. (I need to check but I think a class needs to be specified if the default behavior is not desired.)
- side="front" removed from  < filter class="solr.EdgeNGramFilterFactory" >
 
 
On the VIVO end, runtime.properties (in this example) uses vitro.local.solr.url = http://localhost:8983/solr/collection1.

Originally, in order to get the Solr smoke tests to work, I had to use http://localhost:8983/solr/#/collection1 (i.e. an extra /#/ between solr and the name of the Solr collection). Note the difference between this and the earlier syntax where the collection name did not have to be included and the # was not required.  Without the #, the Solr Smoke tests fail because the tryToConnect method's HttpGet does not return a valid status code.  On the other hand, both http://localhost:8983/solr/#/collection1/admin/ping and  http://localhost:8983/solr/collection1 return valid results for the pinging code in SolrPinger, with the first returning an HTML response and the second returning JSON. Most importantly, if you keep the Solr url as http://localhost:8983/solr/collection1, the search index appears to be indexing new content (it shows a new test faculty member I set up).  I tested this out by turning off the Solr smoke tests in startup listeners and then using the http://localhost:8983/solr/collection1 URL.  

It is also worth nothing that vitro-dependencies uses version 4.10.4 of solrj (the JAVA library used to connect to Solr).  Although indexing seemed to work, it would be useful to experiment with the latest version of Solrj when connecting with Solr 7.0, and in that case, the decision is whether we just to use the latest version or somehow support both the older and newer versions.


To try out with VIVO code, for now (without making changes to VIVO itself), you can use the following steps:

- Get Solr 7.0.4 and install.  For example, say the Solr directory is under /pathtosolr/solr-7.0.4.  Under this directory, you should be able to see the following directories: contrib, dist, docs, example, licenses, server. 
- Copy the directory from this repo into a directory you will have to create in the server directory  (e.g. /pathtosolr/solr-7.0.4/server/collection1 where collection1 is the name of the Solr collection for this example).
- In your VIVO instance, comment out SolrSmokeTests for now by updating the startup_listeners file located here: VIVO\webapp\src\main\webapp\WEB-INF\resources\startup_listeners.txt
- Install VIVO as you normally would, and use http://localhost:8983/solr/collection1 as your vitro.local.solr.url property value in runtime.properties (in VIVO home directory's config directory).  (If your solr uses a port other than 8983, you will have to use that instead.)

Remaining things to explore:
- Updating Solr Smoke Tests to work with the Solr 7 URLs.  Why is HttpGet not working as expected with http://localhost:8983/solr/collection1?
- Exploring the update of the SolrJ dependency from 4 to 7
- Several question to consider, including the all-important world of tests:  Relevancy and ordering? Expected results for a sample set of data?  How does this version compare for the same set of data with the version we had in Solr 4?  Was the behavior in the previous version what we want now?
