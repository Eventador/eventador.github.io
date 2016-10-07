---
layout: default
title: Getting Started Guide
---

# Beta

Eventador.io is currently in Beta. Please submit feedback to [hello@eventador.io](mailto:hello@eventador.io)

# Overview

Eventador is a high performance real-time data pipeline platform based on Apache Kafka. Eventador makes it easy perform analysis and build applications using real time streaming data. Some areas that can benefit from real time data are sensor networks, IoT, click-stream analysis, fraud detection, or anything that requires real-time data. Eventador is deployed to Amazon AWS, and delivered as a service.

Getting started with Eventador takes just a few simple steps.

# Concepts

Eventador facilitates the creation of clusters of cloud servers called **Deployments**. Deployments are built on AWS in a VPC per customer.

Each deployment may have many logical stream namespaces as Kafka **Topics**. Endpoints are exposed in plaintext or SSL for producing and consuming data for each topic.

**Stacks** are interfaces that make consuming or producing data from/to a topic easy. A stack operates on a Topic. Eventador currently is shipping with a default stack running PipelineDB. The PipelineDB stack automatically consumes from the 'defaultsink' topic into the database, allowing for continuous views to be created against the stream.

Each Deployment has a Jupyter **Notebook** server attached to it with helper functions for all endpoints. This enables easy analysis, graphing, data manipulation and reporting.

The Deployment, Topic, and Stacks make a **Data Pipeline**.

# Complete Example

For this example we will do some hypothetical brewery automation based on real time sensor data. Sensors on beer mash tuns gather the current temperature levels and produce it to the pipeline. The data is then aggregated in real-time in PipelineDB. This allows the brewer to monitor the temperature levels and ensure it's below a particular threshold for good beer. If it gets out of threshold an actuator can reduce or increase the temperature.

The complete sample set can be found in our examples repo.

## Prequisites

- Docker

## Step 1: Create an Account

If you don't have one already, [create](http://console.eventador.io/register) an account.

## Step 2: Create a Deployment

You must have at least one deployment.

- Click the [Deployment](http://console.eventador.io/deployments) tab.
- Select the 'Create Deployment' button.
- Name the deployment, and select the compute resource style appropriate for the workload being run.
- Click create. A deployment may take a bit of time to provision. A deployment can not be used until it's status is 'Active' in the [Deployments](http://console.eventador.io/deployments) tab.
- An ACL must be created to allow the producers and consumers to connect. On the [Deployments](http://console.eventador.io/deployments) tab, select the deployment->Security->Add ACL. Add a value in CIDR notation for the IP to whitelist. You can use ```curl ifconfig.co``` to find your IP.

## Step 3: Send some data

Producing data to Eventador is done by sending some data to a Deployment for a particular Topic. In this case, sensors on mash tuns.

- Click the [Deployment](http://console.eventador.io/deployments) tab.
- Select your Deployment
- Select the connections tab.
- Copy/Paste the Kafka Connections.Plain Text field into the example below, and run it on the command line. This will launch a Docker container in the background that's sending temperature data on periodic intervals.

```
docker run -d \
-e EVENTADOR_KAFKA_TOPIC='defaultsink' \
-e EVENTADOR_BOOTSTRAP_SERVERS='<paste kafka connection.plain text here>' \
eventador/brewery_example_producer python /bin/producer.py
```

After a few seconds temperatures will start being produced into the Data Pipeline.

## Step 4: Build some views

Create a continuous view in PipelineDB to perform aggregation functions over time.

```
# create a continuous view as a real-time aggregation
export DBHOST='<paste pipelinedb host here>'
export DBPWD='<paste pipelinedb pwd here>'
docker run -it \
-e PGHOST=$DBHHOST \
-e PGPORT='9000' \
-e PGUSER='defaultsink' \
-e PGPASSWORD=$DBPWD \
eventador/pipelinedb_client \
psql -f /opt/eventador/examples/brewery_example.sql
```

## Step 5: Analyze the data

 Use the Eventador Notebook server for some analysis. Log in, and open the example notebook.

 - Click the [Deployment](http://console.eventador.io/deployments) tab.
 - Select your Deployment
 - Select the connections tab. Click on the notebook link, the notebook username/password is listed right below the link.
- Once in the Eventador Notebook environment, Select File->Open and select the brewery_example.ipynb notebook, and follow the steps in the notebook.

The details of the notebook [here](https://github.com/Eventador/examples/blob/master/notebooks/Brewery%2BExample.ipynb) are on Github.

# Monitoring a Deployment

You can monitor your Deployment via the Eventador [Console](http://console.eventador.io/). This will display a dashboard of statistics from the Kafka nodes within your deployment.

# Software Versions
- Kafka v0.10
- PipelineDB 0.9.3/PostgreSQL 9.5
