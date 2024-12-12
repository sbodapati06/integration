# IBM Event Streams - MQSOURCE Kafka Connector

## 1. Overview

This lab will guide you how to build MQSOURCE Kafka Connector in IBM Event Streams to synchronize/copy data from IBM MQ Queue to IBM Event Streams Topic.

<br>

![](./images/MQSource_Connector_Architecture.png)

<br>

## 2. Create IBM MQ Queue - STUDENT(nn).INVOICES

From the Cloud Pak for Integration Console;<br>
a) Open MQ Console of your Queue Manager<br>
b) Create Queue STUDENT(nn).INVOICES

![](./images/create-queue-1.png)

![](./images/create-queue-2.png)

<br>
<br>

## 3. Create MQSOURCE Connector
In this section you will;<br>
a) Copy/paste the below yaml into OpenShift Console<br>
b) Update the yaml with your IBM Event Streams Topic name, and IBM MQ Queue name<br>
<br>
Logon to the OpenShift Console, and click on the Plus sign (Import Yaml). <br>

![](./images/OCP_Console_PlusSign.png)

Copy/Paste the below KafkaConnector yaml, and update the configuration to match your student number.<br>

```
apiVersion: eventstreams.ibm.com/v1beta2
kind: KafkaConnector
metadata:
  name: student(nn)-mq-source-connector
  namespace: cp4i-eventstreams
  labels:
    # The eventstreams.ibm.com/cluster label identifies the KafkaConnect instance
    # in which to create this connector. That KafkaConnect instance
    # must have the eventstreams.ibm.com/use-connector-resources annotation
    # set to true.
    eventstreams.ibm.com/cluster: my-connect-cluster
spec:
  class: com.ibm.eventstreams.connect.mqsource.MQSourceConnector
  tasksMax: 1
  config:
    topic: STUDENT(nn).INVOICES
    mq.queue.manager: <name-of-your-queue-manager>
    mq.connection.name.list: <name-of-your-queue-manager>-ibm-mq.cp4i-mq.svc(1414)
    mq.channel.name: XXX.SVRCONN
    mq.queue: STUDENT(nn).INVOICES
    mq.user.name: dummy
    mq.password: dummy
    key.converter: org.apache.kafka.connect.storage.StringConverter
    value.converter: org.apache.kafka.connect.storage.StringConverter
    mq.record.builder: com.ibm.eventstreams.connect.mqsource.builders.DefaultRecordBuilder
    mq.message.body.jms: true
```

c) Apply the yaml to "create" Event Streams MQSINK Kafka Connector<br>
![](./images/ocp-console-mqsource-yaml.png)


<br>
<br>

## 4. MQSOURCE Connector Readiness Check

From the OpenShift Console;<br>
a) Navigate to Operators > Installed Operators <br>
b) Change project to cp4i-eventstreams<br>
c) Click on "IBM Event Streams" Operator<br>

![](./images/kafka-connector-1.png)

d) Click on "Kafka Connector" tab <br>

![](./images/kafka-connector-2.png)

Make sure the student(nn)-mq-sink-connector is in Ready state<br>

<br><br>

## 5. Put Sample messages into MQ Queue

From the Cloud Pak for Integration Console;<br>
a) Open MQ Console of your Queue Manager <br>
b) Click on the Manage Tile<br>
c) Click on STUDENT(nn).INVOICES Queue<br>

![](./images/put-to-queue-1.png)

d) Click on "Create" <br>

![](./images/put-to-queue-2.png)

e) Enter test message <br>

![](./images/put-to-queue-3.png)

<br><br>

## 6. Verify Topic data in IBM Event Streams

From the Cloud Pak for Integration Platform Navigator console;<br>
a) Open IBM Event Streams Console <br>
b) Open the Topic STUDENT(nn).INVOICES <br>
C) Verify the messages put into the MQ Queue are moved to the Topic <br>

![](./images/verify-messages.png)


<br><br><br>


### Congratulations !!!