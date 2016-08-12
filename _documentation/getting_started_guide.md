---
layout: post
title: Getting Started Guide
---

# Beta

Eventador.io is currently in Beta. Please submit feedback to [hello@eventador.io](mailto:hello@eventador.io)

# Getting Started

Eventador is a high performance real-time data pipeline based on Apache Kafka. Eventador is deployed to Amazon AWS, and delivered as a service.

Eventador provides Kafka producer and consumer endpoints that speak native Kafka wire protocol and work with a Kafka native drivers.

Eventador also has extended interfaces to make producing and consuming data more powerful.

Getting started with Eventador takes just a few simple steps.

# Creating an account.

To get started you must have an account. [Register here](http://console.eventador.io/register).

# Deployments

A deployment is a group of compute resources assigned to process your data pipeline. A deployment contains a Kafka cluster, zookeeper nodes, has implicit security controls, etc. It's a core construct everything is built on in Eventador. You must have at least one deployment to use the service.

The [Eventador Console](https://console.eventador.io) allows for creation of a deployments.

## Creating a deployment:
- Click the [Deployment](http://console.eventador.io/deployments) tab.
- Select the 'Create Deployment' button.
- Name the deployment, and select the compute resource style appropriate for the workload being run.
- Click create. A deployment may take a bit of time to provision. A deployment can not be used until it's status is 'Active' in the [Deployments](http://console.eventador.io/deployments) tab.
- An ACL must be created to allow the producers and consumers to connect. On the [Deployments](http://console.eventador.io/deployments) tab, select the deployment->ACLS->add ACL. Add a value in CIDR notation for the IP to whitelist.

## Understanding Endpoints
Endpoints are found by selecting [Deployments](http://console.eventador.io/deployments) tab, then connections. There are connection strings for:

- Native Kafka Driver: produce
- Native Kafka Driver: consume
- Additional Interfaces: aka: SQL Interface (PipelineDB): Consume/Produce

These endpoints will be needed to produce to and consume from your new deployment.

# Topics

Eventador allows for full control over Kafka topics. You can create and use topics as you would with any Kafka installation. Currently you manage topics via the native driver interface.

# Producing Data to Eventador

```python
import json
from kafka import KafkaProducer

EVENTADOR_KAFKA_TOPIC = "brewery"  # any topic name, will autocreate if needed
EVENTADOR_BOOTSTRAP_SERVERS = "my bootstrap servers"  # value from deployments tab in UI

payload = {}

# this is the data you want to send in
payload['records'] = [
  {"value": {"sensor": "MashTun1", "temp": 99}},
  {"value": {"sensor": "MashTun2", "temp": 42}}
]

producer = KafkaProducer(value_serializer=lambda v: json.dumps(v).encode('utf-8'),
                         bootstrap_servers=EVENTADOR_BOOTSTRAP_SERVERS)
producer.send(EVENTADOR_KAFKA_TOPIC, json.dumps(payload))
```

# Consuming Data from Eventador

```python
import json
from pprint import pprint
from kafka import KafkaConsumer

EVENTADOR_KAFKA_TOPIC = "brewery"  # any topic name, will autocreate if needed
EVENTADOR_BOOTSTRAP_SERVERS = "my bootstrap servers"  # value from deployments tab in UI

consumer = KafkaConsumer(EVENTADOR_KAFKA_TOPIC, bootstrap_servers=EVENTADOR_BOOTSTRAP_SERVERS)

for msg in consumer:
    print msg
```

# Extended Interfaces

Eventador provides the ability to have extended interfaces. These are additional components provisioned in your deployment that allow for additional functionality and usefulness when building real time data applications.

## SQL Interface

The SQL Interface is based on PipelineDB/PostgreSQL and allows you to build 'continuous views' to aggregate, time-slice, and perform stream processing in real time. The PostgreSQL API provides access to a massive eco-system of SQL compliant tools and drivers. You can build complex programs and algorithms or simply point a reporting tool at the SQL Interface.

## Consuming from the Eventador SQL Interface

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
- PipelineDB 0.9.3/PostgreSQL 9.5
