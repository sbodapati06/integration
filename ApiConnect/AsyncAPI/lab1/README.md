
# IBM Cloud Pak for Integration - Creating AsyncAPI

In this lab, students will go through steps on how to create a Topic in Event Streams, and build an AsyncApi and publish to IBM API Connect Developer Portal and IBM Event Gateway.

<br>

# 1. IBM Event Streams

## Create Topic 

From the Cloud Pak for Integration Platform Navigator, open the IBM Event Streams Console. <br>

Create topic STUDENT00.FLIGHT.LANDINGS. <br>

Use default settings, and create the Topic. <br>

## Generate SCRAM Credentials

![](../images/es-connect-to-cluster.png)

Important: <br>
Save Bootstrap server address into a Notepad <br>

![](../images/es-bootstrap-address.png)

Generate Scram Credentials <br>
![](../images/es-generate-scram.png)

![](../images/es-generate-scram-1.png)
![](../images/es-generate-scram-2.png)
![](../images/es-generate-scram-3.png)
![](../images/es-generate-scram-4.png)

<br>

### Important
Save the username, and password in a Notepad. <br><br>

![](../images/es-generate-scram-5.png)

Download es-cert.pem, es-cert.p12 certificates to Downloads folder. <br>

### Important
Save es-cert password into Notepad.
<br>

![](../images/es-connect-to-cluster-escert-1.png)

The Scram Credentials, and the es-cert.pem certificate will be used during AsyncApi configuration in IBM Api Connect. <br>

<br>
Create JKS file based on es-cert.p12. App Connect KafkaConnector's needs JKS format. <br>

```
cd ~/Downloads 
/usr/bin/keytool -importkeystore -srckeystore es-cert.p12 \
        -srcstoretype PKCS12 \
        -destkeystore es-cert.jks \
        -deststoretype JKS \
	-srcstorepass <password-of-pkcs12-certificate>  \
	-deststorepass passw0rd \
	-noprompt
```

The Scram Credentials, and the es-cert.jks certificate will be used in IBM App Connect Toolkit Flow to simulate flight landing events. <br>

### Important
By now you should have saved the below in Notepad, <br>
a) bootstrap address of your Event Streams Cluster, <br>
b) student00-scram-credentials / and the password <br>
c) es-cert's password <br>
d) es-cert.jks, and es-cert.jks's password (i.e passw0rd) <br><br>


# 2. Api Connect - Manager

## Create AsyncAPI

From the IBM Cloud Pak for Integration Platform Navigator, open the Api Management Console. <br>

From the Home page select "Develop APIs and products" tile.<br>

Add AsyncAPI! <br>
![](../images/apic-add-asyncapi.png)

<br>
Title: student00-asyncapi-flight-landings<br>
Summary: Events emitted when a flight lands<br>
Bootstrap Servers: Copy/Paste the bootstrap address saved above<br>
Topic Name: STUDENT00.FLIGHT.LANDINGS<br>
Security Protocol: SASL_SSL<br>
SASL mechanism: SCRAM-SHA-512<br>
SASL Username: student00-scram-credentials<br>
SASL Password: Copy/Paste the Scram Password saved above<br>
Transport CA Certificate: Drag/Drop the es-cert.pem from the Downloads folder
<br><br>

![](../images/apic-create-api-1.png)
<br><br>
![](../images/apic-create-api-2.png)
<br><br>


## Edit the API

Edit the API to add the Flight Landing Event Schema.<br><br>
Click on the Source Icon as below,
![](../images/apic-edit-api-addschema.png)

Replace "message: {}" line as below <br>
![](../images/apic-edit-api-addschema-2.png) <br>
Double click "message: {}" to select/highlight the entire line. Now, copy and paste the below Avro schema.<br>

```
      message:
        name: landingEvent
        title: Flight landing event
        summary: Flight landing information
        description: >-
          This API provides a stream of events based on the landing of planes at
          Bodapati Airport
        schemaFormat: application/vnd.aai.asyncapi;version=2.0.0
        contentType: application/json
        payload:
          type: object
          required:
            - flight
            - terminal
            - passengers
          properties:
            flight:
              description: ID for the flight that has landed
              type: string
            terminal:
              description: Which terminal the flight has landed at
              type: string
            passengers:
              description: Number of passengers on the flight
              type: string
            additionalProperties: false
        examples:
          - payload: |
              { "flight" : "AI100",  "terminal" : "2", "passengers": "100"}
```
Save the API.<br><br>


## Create Product & Add API


![](../images/apic-create-product.png)

![](../images/apic-create-product-2.png)

![](../images/apic-create-product-3.png)

![](../images/apic-create-product-4.png)

![](../images/apic-create-product-5.png)

![](../images/apic-create-product-6.png)

![](../images/apic-create-product-6.png)

<br>

## Publish the Product
Now, publish the AsyncApi's product to the IBM Api Connect Developer Portal, and to the IBM Event Gateway.

![](../images/apic-stage-product-1.png)

![](../images/apic-stage-product-2.png)

![](../images/apic-stage-product-3.png)

Navigate to API Manager Home (Home Icon on top left) --> Manage Catalogs, select Sandbox Catalog.

Publish the Product to APIC Developer Portal, and Event Gateway.

![](../images/apic-publish-product-1.png)

![](../images/apic-publish-product-2.png)

<br><br>

# 3. API Connect Developer Portal

Locate the developer portal Url, by navigating to API Manager Home (Home Icon on top left) --> Manage Catalogs, select Sandbox Catalog.

Click on "Catalog Settings" tab.
![](../images/apic-dev-portal-navigate.png)

Click on Portal tab on left panel, and copy the Portal URL.
![](../images/apic-dev-portal-navigate.png)


Signon to API Connect Developer Portal

![](../images/apic-dev-portal-signon.png)

![](../images/apic-dev-portal-1.png)


## Subscribe to AsyncAPI

Select student00-asyncapis Product

![](../images/apic-dev-portal-subscribe-1.png)

![](../images/apic-dev-portal-subscribe-2.png)

![](../images/apic-dev-portal-subscribe-3.png)

Important: 
Copy and Save the Key (ClientId), and the Secret into Notepad. These will be used by the Async Client java application.

![](../images/apic-dev-portal-subscribe-4.png)

Select the new application that you just created.
![](../images/apic-dev-portal-subscribe-5.png)

![](../images/apic-dev-portal-subscribe-6.png)

<br><br>

# 4. Event Gateway Bootstrap certificate

Obtain Event Gateway bootstrap certificate, to be used by the Kafka Consumer programs. <br>
<br>

From the command line, logon to OpenShift CLI. <br>

```
oc login --token=sha256~xxxx --server=https://xxxx-coc-cluster:32167
```

```
$ cd ~/Downloads
$ oc project cp4i-apic
$ EVENT_GW_ROUTE_NAME=`oc get route | grep event-gw-client | awk '{print$1}'`
$ BOOTSTRAP_HOST=`oc get route $EVENT_GW_ROUTE_NAME -n cp4i-apic  -o jsonpath="{.spec.host}"`

$ echo -n | openssl s_client -connect $BOOTSTRAP_HOST:443 -servername $BOOTSTRAP_HOST -showcerts | openssl x509 > bootstrap.crt

$ /usr/local/bin/keytool -import -noprompt \
        -alias bootstrapca \
        -file bootstrap.crt \
        -keystore bootstrap.p12 -storepass passw0rd
$ /usr/local/bin/keytool -importkeystore -srckeystore bootstrap.p12 \
        -srcstoretype PKCS12 \
        -destkeystore bootstrap.jks \
        -deststoretype JKS \
	-srcstorepass passw0rd \
	-deststorepass passw0rd \
	-noprompt
```
<br>

Important: <br>
echo $BOOTSTRAP_HOST <br>
(Save BOOTSTRAP_HOST into Notepad) <br>
The BOOTSTRAP_HOST, bootstrap.p12, and password will be used in kafka-sonsole-sonsumer.sh program. <br>
The BOOTSTRAP_HOST, bootstrap.jks, and password will be used in java client application. <br>
<br>


<br>
<br>
<b> END OF LAB </b>
