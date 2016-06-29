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

You may also consume data via the SQL Interface. The SQL Interface is based on PipelineDB/PostgreSQL and allows you to build 'continuous views' to aggregate, query, and perform stream processing in real time. The PostgreSQL API provides access to a massive eco-system of SQL compliant tools and drivers. You can build complex programs and algorithms or simply point a reporting tool at the SQL Interface.

Getting started with Eventador takes just a few simple steps.

## Creating an account.

To get started you must have an account. [Register here](http://console.eventador.io/register).

## Building Pipelines

Pipelines are created on a deployment. So a deployment must first be created for the pipeline. A deployment is a group of AWS compute resources. Multiple pipelines may exist on a single deployment. Deployments are scaled independently of each other. When a pipeline is created, a Kafka topic is created for that pipeline along with all the required plumbing and components.

The [Eventador Console](https://console.eventador.io) allows for creation of a deployments and pipelines.

# Creating a deployment:
- Click the [Deployment](http://console.eventador.io/deployments) tab.
- Select the 'Create Deployment' button.
- Name the deployment, and select the compute resource style appropriate for the workload being run.
- Click create. A deployment may take a bit of time to provision. A deployment can not be used until it's status is 'Active' in the [Deployments](http://console.eventador.io/deployments) tab.
- An ACL must be created to allow the producers to connect. On the [Deployments] tab, select the deployment->ACLS->add ACL. Add in a CIDR notation for the IP to whitelist.

A pipeline can now be created on the deployment.

# Creating a pipeline:
- Click the [Pipelines](http://console.eventador.io/pipelines) tab.
- Select the 'Create Pipeline' button.
- Name the pipeline, give it a description, select the Deployment that was just created, and give a user/password for the data store.
- Click create. A pipeline is now created and can be seen under the [Pipelines](http://console.eventador.io/pipelines) tab.

# Understanding Endpoints
Endpoints are found by selecting [Pipelines](http://console.eventador.io/pipelines) tab, then the pipeline, then connections. There are connection strings for:

- Pipeline REST Interface: Produce
- Pipeline Rest Interface: consume
- SQL Interface (PipelineDB): Consume

These endpoints will be needed to produce and consume data from your new pipeline.

## Publishing Data to the Eventador Pipeline

Publishing data to the Eventador Pipeline is done via the REST endpoint. It's important to note that a schema must be defined for the Pipeline before data can be sent to it. In this case we are using serializing data to Apache Avro.

The examples below assume python is installed on your system.

# Creating a schema

```python
import json
import requests
from pprint import pprint

username = "myusername" # change me to value in console->pipeline->connections
endpoint = "xxxxxx" # change me to value in console->pipeline->connections
schema = "brewery"  # change me to the pipeline name    
namespace = "{}_{}".format(username, schema)
uri = "https://schema-registry.{}.vip.eventador.io/subjects/{}-value/versions".format(endpoint, namespace)

payload = {}
payload['schema'] = """
  {"type": "record",
   "name": "brewery2",
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

This code will return a schema ID value. This is used when sending data into the pipeline.


# Sending data to Eventador Pipeline
```python
import json
import requests

username = "myusername" # change me to value in console->pipeline->connections
endpoint = "xxxxxx" # change me to value in console->pipeline->connections
schema = "brewery"  # change me to the pipeline name   
namespace = "{}_{}".format(username, schema)
schema_id = "52" # change to the value returned from the previous step
uri = "https://api.{}.vip.eventador.io/topics/{}".format(endpoint, namespace)

payload = {}

# this is the ID for the schema to use, it was returned in the previous step
payload['value_schema_id'] = "{}".format(schema_id)

# this is the data being sent in
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

# Registering a consumer

First we must register a consumer. This ensures Kafka understands state as data is consumed.

```bash
curl -s -X POST -H "Content-Type: application/vnd.kafka.v1+json" \
--data '{"format": "avro", "auto.offset.reset": "smallest"}' \
https://api.xxxxxx.vip.eventador.io/consumers/my_consumer

{
  "instance_id": "rest-consumer-1",
   "base_uri": "https://api.xxxxxx.eventador.io/consumers/my_consumer/instances/rest-consumer-1"
}

```

Next consume the data using the URI that is returned when we create a consumer.

```bash
curl -s -X GET -H "Accept: application/vnd.kafka.avro.v1+json" \
https://api.xxxxxx.eventador.io/consumers/my_consumer/instances/rest-consumer-1/topics/myusername_brewery
[
  {
     "key": null,
     "value": {"sensor": "MashTun1", "temp": 28},
     "partition": 0,
     "offset": 0
   },
   {
     "key": null,
     "value": {"sensor": "MashTun2", "temp": 27},
     "partition": 0,
     "offset": 1
   }
]
```

# Consuming from the Eventador SQL Interface

The SQL Interface is based on PipelineDB/PostgreSQL. You can define a continuous view using simple SQL syntax and the views are continuously updated as data comes in from the pipeline. Views can be simple aggregations, time-windows, or anything else as defined by the PipelineDB SQL syntax and functions.

A continuous view is a view of a SQL Stream. The stream is automatically built when a pipeline is created and has a sample continuous view created on it. You can create continuous views as needed. A sample view is created named ```ev_sample_view``` and is available in the users database. The database enforces SSL and causes the client to use SSL by default.

To login to the database and query the sample view and create more continuous views:

- Download the PipelineDB client [here](https://www.pipelinedb.com/download).
- Connect to the database using psql with your username, database. The login information, and hostname is available in the Eventador [Console](http://console.eventador.io/pipelines), select the pipeline then connections.
- By connvention, the database username is your login username, and the database_name is username_pipelinename.

```bash
psql -U <username> -h <hostname> -p 9000 <database_name>
```

Query the sample view:

```sql
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

More information on continuous views is available in the [PipelineDB documentation](http://docs.pipelinedb.com/continuous-views.html)

## Monitoring the pipeline

You can monitor your pipeline via the Eventador [Console](http://console.eventador.io/pipelines). Click on the pipeline to monitor. The statistics (default) tab shows some metrics about the pipeline.

## Software Versions
- Kafka v0.10
- Confluent kafka-REST proxy v3.0.0
- Confluent Schema Registry v3.0.0
- PipelineDB 0.9.3/PostgreSQL 9.5
