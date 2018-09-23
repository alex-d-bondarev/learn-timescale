learn-timescale
===============

[Presentation file](https://docs.google.com/presentation/d/1rFy5cCkXF7AUrnD7SnCHWXU8N2rzUkZcbHb_335_JR4/edit?usp=sharing)

Goal
----

Check whether InfluxDB has such significant write throughput difference with Elasticsearch as reported on official Influx Data [website](https://www.influxdata.com/time-series-database/).
Also check other DBs to improve comparison results

Rules: 
1. Use write commands that are easy to find.
1. Do not search for ways to optimize requests, but use the ones from tutorials.
1. Use the same machine for all measurements.
1. Reboot machine before each measurement. 

Plan
----

1. Pull **InfluxDB** docker image
1. Insert data and measure results
1. Pull **Elasticsearch** docker image 
1. Insert data and measure results
1. Add results to [my presentation file](https://docs.google.com/presentation/d/1rFy5cCkXF7AUrnD7SnCHWXU8N2rzUkZcbHb_335_JR4/edit?usp=sharing)
1. Add _steps to reproduce_ into this README file. 

Insertion details
-----------------

Insert should be the following:
1. Data base name/index = mydb
1. Measurement/type/table = kinda
1. Tag = test
1. Thread = thread number \[1; 4\]
1. RandomValue = random number \[0; 1000\]
1. Timestamp = timestamp

Steps to reproduce
------------------

### InfluxDB

1. Run the following in terminal:
    ```
    # Pull image
    docker pull influxdb:1.6.3
                
    # Launch InfluxDB
    docker run -p 8086:8086 \
          -v $PWD:/var/lib/influxdb \
          influxdb:1.6.3
          
    # Create DB
    # curl -G http://localhost:8086/query --data-urlencode "q=CREATE DATABASE mydb"
    # that one was deprecated :(
    # Try this one:
    curl -XPOST http://localhost:8086/query --data-urlencode "q=CREATE DATABASE mydb"
        
    # Check that write functionality is working
    curl -i -XPOST 'http://localhost:8086/write?db=mydb' \
            --data 'kinda,tag=test,thread=1 randomValue=6 946684800'
        
    # Check write command results
    curl -G 'http://localhost:8086/query?db=mydb' --data-urlencode 'q=SELECT * FROM "kinda"'
    # {"results":[{"statement_id":0,"series":[{"name":"kinda","columns":["time","randomValue","tag","thread"],"values":[["1970-01-01T00:00:00.9466848Z",6,"test","1"]]}]}]}
        
    # Clear DB
    curl -i -XPOST 'http://localhost:8086/query' -d db=mydb -d q='DELETE from "kinda"'
    ```
1. Open ./jmeter/InfluxDB_Test_Plan.jmx via jMeter
1. Run it.
1. Check results in listeners (e.g. `Response Time Graph`)
1. Clear DB:
    ```
    # Clear DB
    curl -i -XPOST 'http://localhost:8086/query' -d db=mydb -d q='DELETE from "kinda"'
    ```
1. Stop Docker by terminating `docker run...`

### Elasticsearch

1. Run the following in terminal:
    ```
    # Pull image
    docker pull docker.elastic.co/elasticsearch/elasticsearch:6.3.2
                
    # Launch Elasticsearch
    docker run -p 9200:9200 -p 9300:9300 \
            -e "discovery.type=single-node" \
            docker.elastic.co/elasticsearch/elasticsearch:6.3.2
          
    # Check that write functionality is working
    curl -XPOST http://localhost:9200/mydb/kinda \
            -H 'Content-Type: application/json' \
            --data '{"tag":"test", "thread":1, "randomValue":6, "timestamp":946684800}'
        
    # Check write command results
    curl -G 'http://localhost:9200/_search?q=test'
    # {"took":129,"timed_out":false,"_shards":{"total":5,"successful":5,"skipped":0,"failed":0},"hits":{"total":1,"max_score":0.2876821,"hits":[{"_index":"mydb","_type":"kinda","_id":"0EwFAmYBg5Z2-3YbKeuu","_score":0.2876821,"_source":{"tag":"test", "thread":1, "randomValue":6, "timestamp":946684800}}]}}
        
    # Clear DB
    curl -XDELETE localhost:9200/*
    ```
1. Open ./jmeter/Elasticsearch_Test_Plan.jmx via jMeter
1. Run it.
1. Check results in listeners (e.g. `Response Time Graph`)
1. Clear DB:
    ```
    # Clear DB
    curl -XDELETE localhost:9200/*
    ```
1. Stop Docker by terminating `docker run...`
