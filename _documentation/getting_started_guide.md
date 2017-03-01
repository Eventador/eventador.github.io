---
layout: default
title: Getting Started Guide
---

# Overview

Eventador is a high performance real-time data pipeline platform based on Apache Kafka. Eventador makes it easy to perform analysis and build applications using real time streaming data. Some areas that can benefit from real time data are sensor networks, IoT, click-stream analysis, fraud detection, or anything that requires real-time data. Eventador is deployed to Amazon AWS, and delivered as a service.

Getting started with Eventador takes just a few simple steps. But first, some concepts.

# Concepts

Eventador.io is a real-time streaming data platform. We enable you to easily build data pipelines for any real-time data use case from applications to research.

The core component of Eventador.io is a deployment. A deployment is a logical grouping of components that make up your data pipeline. This includes an Apache Kafka cluster with consume and produce endpoints, A PrestoDB endpoint, Eventador Notebook endpoints and so on.

Deployments contain topics. Multiple streams of data can exist within a single deployment. In general it's best to separate topics based on some construct of the use-case the data stream is being used for. Perhaps one for marketing data and one for product suggestions for instance.

A deployment consists of:

- A Kafka cluster including Zookeeper nodes (the backbone of the service)
- A PrestoDB cluster (for quick analysis and reporting) (developer plan or above)
- A Eventador Notebook server (for analysis, experiments, and output) (developer plan or above)

When you sign up you get a distinct and isolated VPC that your deployments live in. You **must** grant access to each deployment via the deployments->Security tab in order for any IP traffic to be allowed through. More on this below.

# Quickstart Example

For this example we will do some hypothetical brewery automation based on real time sensor data. Sensors on beer mash tuns gather the current temperature levels and produce it to the pipeline. The data can then be queried right from the Kafka via SQL. This allows the brewer to monitor the temperature levels and ensure it's below a particular threshold for good beer. If it gets out of threshold an actuator can reduce or increase the temperature.

The complete sample set can be found in our examples repo.

## Prequisites

- [kafkacat](https://github.com/edenhill/kafkacat)

## Step 1: Create an Account

If you don't have one already, [create](http://console.eventador.io/register) an account.
If you don't already have a credit card on file, enter one in the [accounts](http://console.eventador.io/account) page.

## Step 2: Create a Deployment

You must have at least one deployment.

- Click the [Kafka Deployment](http://console.eventador.io/deployments) tab.
- Select the 'Create Kafka Deployment' button.
- Select a plan that fits your workload. For most all workloads this is a developer plan or above.
- Name the deployment, and select the compute resource style appropriate for the workload being run.
- Click create. A deployment may take a bit of time to provision. A deployment can not be used until it's status is 'Active' in the [Deployments](http://console.eventador.io/deployments) tab.
- An ACL must be created to allow the producers and consumers to connect. On the [Kafka Deployments](http://console.eventador.io/deployments) tab, select the deployment->Security->Add ACL. Add a value in CIDR notation for the IP to whitelist. You can use ```curl ifconfig.co``` to find your IP.

## Step 3: Create a Topic

A topic is a container for a stream of data pertaining to some use case.

- Click the [Kafka Deployment](http://console.eventador.io/deployments) tab.
- Select the Topics tab.
- Select the 'Add Topic' button
- Name the topic 'brewery'
- 32 partitions, replication factor 3
- Click create

## Step 4: Send some data

Producing data to Eventador is done by sending some data to a Kafka Deployment for a particular Topic. In this case, hypothetical data on sensors for mash tuns. We are going to use the kafkacat utility to send data from the command line, but it could be any client using any Kafka driver.

- Click the [Kafka Deployment](http://console.eventador.io/deployments) tab.
- Select your Kafka Deployment.
- Select the connections tab.
- Copy/Paste the Kafka Connections.Plain Text field into the example below.

```
BROKERS=<the value pasted from console>
echo '{"name": "mashtun01", "temp": "38"}' | kafkacat -P -b $BROKERS -t brewery
echo '{"name": "mashtun02", "temp": "37"}' | kafkacat -P -b $BROKERS -t brewery
echo '{"name": "mashtun01", "temp": "37"}' | kafkacat -P -b $BROKERS -t brewery
echo '{"name": "mashtun03", "temp": "44"}' | kafkacat -P -b $BROKERS -t brewery
```

## Step 5: Consume some data

Eventador.io has a number of endpoints where data can be consumed depending on your use case.

- Raw Kafka message
- PrestoDB SQL (developer plan and above)
- Eventador Notebook (via SQL or via Python or R or Julia) (developer plan and above)

### Consuming data via Kafka

```
BROKERS=<the value pasted from console>
kafkacat -C -b $BROKERS -t brewery
```
### Consuming data via PrestoDB SQL (Developer Plan and above)

In this case let's assume you want to consume the messages to create a report in PrestoDB SQL, perhaps for a report to the brewer. In this case we are pushing a JSON object into the data pipeline, so we will use JSON operators in PrestoDB to access those fields.

- Click on the [deployment](http://console.eventador.io/deployments) tab.
- Choose the deployment you created in step 2.
- Select the SQL tab
- Run the following SQL in the SQL pane:

```SQL
SELECT
json_extract(_message, '$.name') as sensor_name,
round(avg(try_cast(json_extract(_message, '$.temp') as integer))) as avg_temp
FROM brewery
GROUP BY 1
ORDER BY 2;

sensor_name | avg_temp
----------------------
"mashtun02" | 37
"mashtun01" | 38
"mashtun03" | 44
```

# Monitoring a Deployment

You can monitor your Kafka Deployment via the Eventador [Console](http://console.eventador.io/). This will display a dashboard of statistics from the Kafka nodes within your deployment.

# Software Versions
- Kafka v0.10.1
- PrestoDB 1.66
