---
layout: post
title: Getting Started Guide
---

## Beta

Eventador.io is currently in Beta. Please submit feedback to [hello@eventador.io](mailto:hello@eventador.io)

## Getting Started

Eventador is a high performance real-time data pipeline based on Apache Kafka. Eventador is deployed to Amazon AWS, and delivered as a service.

Eventador provides a producer and consumer interfaces. It also provides an SQL Interface in the form of PipelineDB/PostgreSQL. Other Interfaces will be added in the future.

You produce data to Eventador using a REST interface, and consume data via the same REST interface. The REST interface also provides control over Kafka topics and schema. You can create multiple pipelines to form more complex event processing systems.

You may also consume data via the SQL Interface. The SQL Interface is based on PipelineDB/PostgreSQL and allows you to build 'continuous views' to aggregate, time-slice, and perform stream processing in real time. The PostgreSQL API provides access to a massive eco-system of SQL compliant tools and drivers. You can build complex programs and algorithms or simply point a reporting tool at the SQL Interface.

Getting started with Eventador takes just a few simple steps.

## Creating an account.

To get started you must have an account. [Register here](http://console.eventador.io/register).

## Building Pipelines

Pipelines are created on a deployment. A deployment must first be created for the pipeline. A deployment is a group of AWS compute resources. Multiple pipelines may exist on a single deployment. Deployments are scaled independently of each other. When a pipeline is created, a Kafka topic is created for that pipeline along with all the required plumbing and components.

The [Eventador Console](https://console.eventador.io) allows for creation of a deployments and pipelines.

# Creating a deployment:
- Click the [Deployment](http://console.eventador.io/deployments) tab.
- Select the 'Create Deployment' button.
- Name the deployment, and select the compute resource style appropriate for the workload being run.
- Click create. A deployment may take a bit of time to provision. A deployment can not be used until it's status is 'Active' in the [Deployments](http://console.eventador.io/deployments) tab.
- An ACL must be created to allow the producers to connect. On the [Deployments](http://console.eventador.io/deployments) tab, select the deployment->ACLS->add ACL. Add a value in CIDR notation for the IP to whitelist.

A pipeline can now be created on the deployment.

# Creating a pipeline:
- Click the [Pipelines](http://console.eventador.io/pipelines) tab.
- Select the 'Create Pipeline' button.
- Name the pipeline, give it a description, select the Deployment that was just created, and give a user/password for the data store.
- Click create. A pipeline is now created and can be seen under the [Pipelines](http://console.eventador.io/pipelines) tab.

# Understanding Endpoints
Endpoints are found by selecting [Pipelines](http://console.eventador.io/pipelines) tab, then the pipeline, then connections. There are connection strings for:

- Pipeline REST Interface: produce
- Pipeline Rest Interface: consume
- SQL Interface (PipelineDB): consume

These endpoints will be needed to produce and consume data from your new pipeline.

## Publishing Data to the Eventador Pipeline

Publishing data to the Eventador Pipeline is done via the REST endpoint. It's important to note that a schema must be defined for the Pipeline before data can be sent to it. The examples below assume python is installed on your system.

# Creating a schema

```python
import json
import requests
from pprint import pprint

username = "myusername" # change me to value in console->pipeline->connections
endpoint = "xxxxxx" # change me to value in console->pipeline->connections
pipeline = "brewery"  # change me to the pipeline name    
topic = "{}_{}".format(username, pipeline)
uri = "https://schema-registry.{}.vip.eventador.io/subjects/{}-value/versions".format(endpoint, topic)

# schema is two values: sensor (string), temp (int)
payload = {}
payload['schema'] = """
  {"type": "record",
   "name": "mybreweryschema",
   "fields": [
      {"name": "sensor", "type": "string"},
      {"name": "temp", "type": "int"}
]}
"""

headers = {'Content-Type': 'application/vnd.schemaregistry.v1+json'}
r = requests.post(uri,
                  data=json.dumps(payload),
                  headers=headers)

pprint(r.json())
```

This code will return a schema ID. This value is used when sending data into the pipeline.


# Sending data to Eventador Pipeline
```python
import json
import requests

username = "myusername" # change me to value in console->pipeline->connections
endpoint = "xxxxxx" # change me to value in console->pipeline->connections
pipeline = "brewery"  # change me to the pipeline name   
topic = "{}_{}".format(username, pipeline)
schema_id = "52" # change to the value returned from the previous step
uri = "https://api.{}.vip.eventador.io/topics/{}".format(endpoint, topic)

payload = {}

# this is the ID for the schema to use, it was returned in the previous step
payload['value_schema_id'] = "{}".format(schema_id)

# this is the data you want to send in
payload['records'] = [
  {"value": {"sensor": "MashTun1", "temp":99}},
  {"value": {"sensor": "MashTun2", "temp":42}}
]

headers = {'Content-Type': 'application/vnd.kafka.avro.v1+json'}
r = requests.post(uri,
                  data=json.dumps(payload),
                  headers=headers)
print r.json()

```

More information on the REST interface can be [found here](http://docs.confluent.io/3.0.0/kafka-rest/docs/api.html).

## Consuming Data from Eventador

Data can be consumed from Eventador in two ways. It can be directly consumed from the Eventador Pipeline REST interface, or it can be consumed using the SQL Interface (PipelineDB).

# Consuming from the Eventador Pipeline using the REST interface

Consuming data via the REST interface requires two steps. First registering a consumer, then consuming from the pipeline.

```python
import json
import requests
import time
from pprint import pprint

username = "myusername" # change me to value in console->pipeline->connections
endpoint = "xxxxxx" # change me to value in console->pipeline->connections
pipeline = "brewery"  # change me to the pipeline name   
topic = "{}_{}".format(username, pipeline)
consumer_group = "my_group_of_application_servers" # logical group of consumers sharing offsets
consumer_uri = "https://api.{}.vip.eventador.io/consumers/{}".format(endpoint, consumer_group)

# register a new consumer (node)
register_consumer = {"format": "avro",
                     "auto.offset.reset": "largest" # start at most recent offset,
                                                    # can also use 'smallest',
                                                    # or omit entirely
                    }

r = requests.post(consumer_uri, data=json.dumps(register_consumer),
        headers={'Content-Type': 'application/vnd.kafka.v1+json'})

# this will contain our assigned endpoint to read messages from
base_uri = r.json()['base_uri']
print("Using endpoint: {}".format(base_uri))

# loop while polling for new messages
topic_uri = "{}/topics/{}".format(base_uri, topic)
while True:
    r = requests.get(topic_uri,
            headers={'Accept': 'application/vnd.kafka.avro.v1+json'})
    print r.json()
    time.sleep(1)
```

# Consuming from the Eventador SQL Interface

The SQL Interface is based on PipelineDB/PostgreSQL. You can define a continuous view using simple SQL syntax and the views are continuously updated as data comes in from the pipeline. Views can be simple aggregations, time-windows, advanced analytics, or anything else as defined by the PipelineDB SQL syntax and functions.

A continuous view is a view of a SQL Stream. The stream is automatically built when a pipeline is created. A sample continuous view is created named ```ev_sample_view```. You can create more continuous views as needed. The database enforces SSL and causes the client to use SSL by default.

To login to the database and query the sample view and create more continuous views:

- Download the PipelineDB client [here](https://www.pipelinedb.com/download).
- Connect to the database using psql with your username, database. The login information, and hostname is available in the Eventador [Console](http://console.eventador.io/pipelines), select the pipeline then connections.
- By convention, the database username is your login username, and the database_name is username_pipelinename.

```bash
psql -U <username> -h <hostname> -p 9000 <database_name>
```

Query the sample view:

```sql
-- just select some basic data
SELECT * FROM ev_sample_view LIMIT 10;
```

Continuous views are created on a stream. Every pipeline has a default stream named ```<pipeline name>_stream``` created automatically, with a payload field with the data type JSON.

You can create a new continuous view:

```sql
-- average temperature of sensors over the last 5 minutes by sensor name
CREATE CONTINUOUS VIEW brewery_sensor_temps WITH (max_age = '5 minutes') AS
SELECT payload->>'sensor', AVG((payload->>'temp'::text)::numeric)
FROM brewery_stream
GROUP BY payload->>'sensor';
```

or

```sql
-- average temperature by hour of day
CREATE CONTINUOUS VIEW brewery_temps_by_hourofday AS
SELECT date_part('hour', arrival_timestamp) as ts,
avg((payload->>'temp'::text)::numeric)
FROM brewery_stream
GROUP BY ts;
```

More information on continuous views is available in the [PipelineDB documentation](http://docs.pipelinedb.com/continuous-views.html)

## Monitoring the pipeline

You can monitor your pipeline via the Eventador [Console](http://console.eventador.io/pipelines). Click on the pipeline to monitor. The statistics (default) tab shows some metrics about the pipeline.

## Software Versions
- Kafka v0.10
- Confluent kafka-REST proxy v3.0.0
- Confluent Schema Registry v3.0.0
- PipelineDB 0.9.3/PostgreSQL 9.5
