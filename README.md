Time zones of the LOD-cloud SPARQL endpoints
============================================

> SPARQL guessing of endpoint Earth locations

Idea
----

In this repository, we propose an unusual SPARQL-based method to
"guesstimate" the real location of an endpoint server on Earth. In
order to demonstrate it, we apply it on the LOD- cloud.

Practically, our method relies on the use of a specific (and not very
common) SPARQL 1.1 built-in function: NOW() whose result is the
current time of the local machine executing the SPARQL query _i.e._
the time of the server running the SPARQL endpoint itself. To retrieve
such information, we use the following query:

    SELECT ?time ?tz
    WHERE {
      VALUES ?foo {"bar"}        # Force the declaration of a variable.
      BIND ( NOW() AS ?time )    # Gets the server current time.
      BIND ( TZ (?time) AS ?tz ) # Tries to extract the time zone.
    }

The result of such a query may help someone to have an idea of the
location on Earth of the SPARQL endpoint, as the time gives an
indication about the time zone.

In order to validate our approach, we tested it using the resources
provided by the Linked Open Data Cloud <https://lod-cloud.net/>. In
particular, we retrieve all their available access URLs of SPARQL
endpoints and sent our query to them.


Repository files
----------------

This repository contains:

- `README.md` (this file): providing documentation;
- `LICENSE`;
- `lod-data.json`: the raw JSON data about the LOD-cloud resources;
- `list_of_endpoint_UP.txt`: the list of accessible SPARQL endpoints in the cloud considered as UP by the LOD maintainers;
- `list_of_unique_endpoint_UP.txt`: a refined list of the former where duplicates have been removed;
- `time-zone-logs.txt`: the result file obtained after running sending the same aforementioned query to each SPARQL endpoint.

Note: all the experiments (downloads and queries)  were run on Monday 7th March 2022.


Commands
--------

The following _bash_ commands were run to obtain the files listed
above. They are making use of [jq](https://stedolan.github.io/jq/) a
lightweight and flexible command-line JSON processor.

    # General information about the available SPARQL endpoints in the LOD-cloud.
    cat lod-data.json | jq '..|.sparql?|select( . != null )' | grep -c access_url
    cat lod-data.json | jq '..|.sparql?|select( . != null )' | grep -c OK
    cat lod-data.json | jq '..|.sparql?|select( . != null )' | grep -c FAIL

    # Extract the list of access URL.
    cat lod-data.json | jq '..|.sparql?|select( . != null )' | jq 'select( .[0].status == "OK")' | grep access_url | awk -F '"access_url": "|",' '{print $2}' > list_of_endpoint_UP.txt
    
    # Extract the list of access URL, sort it and remove duplicates.
    cat lod-data.json | jq '..|.sparql?|select( . != null )' | jq 'select( .[0].status == "OK")' | grep access_url | awk -F '"access_url": "|",' '{print $2}' | sort -u > list_of_unique_endpoint_UP.txt
    
    # Send the query to each endpoint.
    time cat list_of_unique_endpoint_UP.txt | \
       while read -r url; do \
          echo -e "++++++++++\n++++++++++\n$url\n$(date)\n" ; \
	  curl --data-urlencode "query=select ?time ?tz where { values ?x {"1"} bind ( now() as ?time ) bind ( tz(?time) as ?tz )}" $url ; \
	  echo -e "\n\n" ; \
	  sleep 1 ; \
       done > time-zone-logs.txt

    # To have an idea of the number of successful queries
    grep "2022-03" time-zone-logs.txt


Author
------

Damien Graux <https://dgraux.github.io/>  
Inria, France  
2022
