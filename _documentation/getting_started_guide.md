---
layout: post
title: Getting Started Guide
---

## Getting Started

Eventador is a high performance real-time data pipeline based on Apache Kafka. Eventador is deployed to Amazon AWS, and delivered as a service.

Eventador provides a producer and consumer interfaces. It also provides an Aggregation Interface in the form of PipelineDB/PostgreSQL. Other Interfaces will be added in the future.

You produce data to Eventador using a REST interface, and consume data via the same REST interface. The REST interface also provides control over Kafka topics and schema. You can create multiple pipelines to form more complex event processing systems.

You may also consume data via the Aggregation Interface. The Aggregation Interface is based on PipelineDB/PostgreSQL and allows you to build 'continuous views' to aggregate, query, and perform stream processing in real time. The PostgreSQL API provides access to a massive eco-system of SQL compliant tools and drivers. You can build complex programs and algorithms or simply point a reporting tool at the Aggregation Interface.

Getting started with Eventador takes just a few simple steps.

## Creating an account.

To get started you must have an account. [Register here](http://console.eventador.io/register).

## Building Pipelines

Pipelines are created on a deployment. So a deployment must first be created for the pipeline to reside on. A deployment is a group of AWS compute resources under the pipeline. Multiple pipelines may exist on a deployment. Deployments are scaled independently of each other.

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

## Publishing Data to the Eventador Pipeline

```
curl -X POST -H "Content-Type: application/vnd.kafka.avro.v1+json" \
      --data '{"value_schema": "{\"type\": \"record\", \"name\": \"User\", \"fields\": [{\"name\": \"name\", \"type\": \"string\"}]}", "records": [{"value": {"name": "testUser"}}]}' \
      "http://localhost:8082/topics/avrotest"
  {"offsets":[{"partition":0,"offset":0,"error_code":null,"error":null}],"key_schema_id":null,"value_schema_id":21}
```

# Publishing data to the pipeline

Once a schema is created users can publish data to the REST endpoint.

```
curl -X POST -H "Content-Type: application/vnd.kafka.avro.v1+json" \
      --data '{"value_schema": "{\"type\": \"record\", \"name\": \"User\", \"fields\": [{\"name\": \"name\", \"type\": \"string\"}]}", "records": [{"value": {"name": "testUser"}}]}' \
      "http://localhost:8082/topics/avrotest"
  {"offsets":[{"partition":0,"offset":0,"error_code":null,"error":null}],"key_schema_id":null,"value_schema_id":21}
```

## Consuming Data from Eventador

Data can be consumed from Eventador in two ways. It can be directly consumed from the Kafka REST interface, or it can be consumed using the Aggregation Interface.

# Consuming from the Eventador Pipeline

Consuming data via the REST interface:

```
curl -X POST -H "Content-Type: application/vnd.kafka.v1+json" \
      --data '{"id": "my_instance", "format": "avro", "auto.offset.reset": "smallest"}' \
      http://localhost:8082/consumers/my_avro_consumer
{"instance_id":"my_instance","base_uri":"http://localhost:8082/consumers/my_avro_consumer/instances/my_instance"}
```

# Consuming from the Eventador Aggregation Interface

The Aggregation Interface is based on PipelineDB/PostgreSQL. You can define a continuous view using simple SQL syntax and the views are continuously updated as data comes in from the pipeline. Views can be simple aggregations, time-windows, or anything else as defined by the PipelineDB syntax.

A continuous view is a view of a SQL Stream. The stream is automatically built when a pipeline is created and has a sample continuous view created on it. You can create continuous views as needed. The continuous view is named ```ev_sample_view``` and is available in the users database. The database enforces SSL and causes the client to use SSL by default.

To login to the database and query the sample view and create more continuous views:

- Download the PipelineDB client [here](https://www.pipelinedb.com/download).
- Connect to the database using psql with your username, database. The login information, and hostname is available in the Eventador Console at ```http://console.eventador.io/pipeline_detail/<pipeline_name>```.
- The username is ```login name```, the database name is ```username_pipelinename```

```
psql -U <username> -h <hostname> -p 9000 <database name>
```

Query the sample view:

```
SELECT * FROM ev_sample_view LIMIT 10;
```

Continuous views are created on a stream. Every pipeline has a default stream named ```<pipeline name>_stream``` created automatically, with a payload field with the data type JSON.

You can create a new continuous view:

```
CREATE CONTINUOUS VIEW sensor_temps WITH (max_age = '5 minutes') AS
   SELECT payload->>sensor::integer, AVG(payload->>temp::numeric)
   FROM sensor_stream
GROUP BY payload->>sensor;
```

More information on continuous views is available in the [PipelineDB documentation](http://docs.pipelinedb.com/continuous-views.html)

## Monitoring the pipeline

You can monitor your pipeline via the Eventador Console. From the [pipelines](http://console.eventador.io/pipelines) click on the pipeline to monitor. The statistics (default) tab shows some metrics about the pipeline.

## Versions
- Kafka v0.10
- Confluent kafka-REST proxy v3.0.0
- Confluent Schema Registry v3.0.0
- PipelineDB 0.9.3/PostgreSQL 9.5
