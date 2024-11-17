# IBM Cloud Pak for Integration - AsyncApi

## Consuming Flight Landing Events from Event Gateway through App Connect Toolkit Flow

In this lab, you will consume the flight landing events via the Event Endpoint Management Gateway using an App Connect Toolkit Flow.

<br>
<b>Note: Please finish lab1 & lab2 before attempting this lab.

<br>

## 1. App Connect Toolkit
Open the App Connect Toolkit. <br>

<br>

### Create Kafka Policy Project

![](./images/images/ace-toolkit-policy-1.png)

<b>Project Name: <u>eem-policy-project</u>

<br>

#### Create New Policy
![](./images/images/ace-toolkit-policy-2.png)


<b> Policy Name: dev <br>
<br>
Fill in the Policy Template as below; <br>
<b> Type: kafka <br>
Template: kafka <br>
Bootstrap Servers: BOOTSTRAP_HOST:443 (event gateway bootstrap host from lab1) <br>
Security Protocol: SASL_SSL <br>
SASL Mechanism: PLAIN <br>
SSL Protocol: TLSv12 <br>
Security Identity (DSN): eem-security-identity <br>
SASL Config: org.apache.kafka.common.security.plain.PlainLoginModule required; <br>
SSL Truststore location: /home/aceuser/truststores/eem-truststore.jks <br>
SSL truststore type: JKS <br>
SSL truststore identity: eem-truststore-password <br>
</b>
![](./images/images/ace-toolkit-policy-5.png)
 
<br>
Control-S to Save the policy file.

<br>

#### <b> Export Policy as Archive (zip) file </b>

![](./images/images/ace-toolkit-policy-3.png)

Export to Downloads folder, name it <u>eem-policy.zip </u><br>

![](./images/images/ace-toolkit-policy-4.png)

<br> 

### Create New Application
<b>Name: Flight_Landing_Events_Consumer_Application <br>
![](./images/images/ace-toolkit-app-1.png)

<br>

#### Create Message Flow
![](./images/images/ace-toolkit-app-2.png)


#### Build the Flow
Now, from the <u>Connectors</u> --> <u>Kafka</u> Palette, drag KafkaConsumer Node into the Design area. <br>
Similary, from the <u>Connectors</u> --> <u>File</u> Palette drag FileOutput Node from the . <br>
<br>
Connect the Out Terminal of KafkaConsumer Node to the In Terminal of FileOutput Node. <br>

![](./images/images/ace-toolkit-app-3.png)

#### Update KafkaConsumer Node Settings
![](./images/images/ace-toolkit-app-4.png)

Client ID: <br>
![](./images/images/ace-toolkit-app-8.png)
Logon to the APIC Developer Portal to obtain client_id as in above. This is the client_id of the AsyncApi/subscribe operation. You can find this in APIC Developer portal > Your Async API > subscribe operation. Look at the properties section.<br>

Control-S to Save the the flow. <br>
<br>

#### FileOutput Node Settings
![](./images/images/ace-toolkit-app-5.png)

<br>

### Create BAR File
Right click on Flight_Landing_Events_Consumer_Application > new > Bar file, as below.<br>
Give it a name, and click "Finish".<br>
![](./images/images/ace-toolkit-app-6.png)

Under the "Prepare" tab, Select Flight_Landing_Events_Consumer_Application, and click "Build and Save" button. <br>
![](./images/images/ace-toolkit-app-7.png)


<br>

## 2. App Connect Dashboard

Navigate to the Cloud Pak for Integration Platform Navigator Console, and open App Connect Dashboard or Runtime Console.<br>
<br>

### Create Configurations
<br>
In this section you will create; <br>
(1) PolicyProject for eem-policy.zip <br>
(2) TrustStore for the Event Gateway Certificate.<br>
(3) setdbparms.txt for storing the credentials of the APIC Portal Subscribing Application. and password of the bootstrap.jks<br>
<br>
Click on "Create Configuration" <br>
Select "Policy Project", and drag & drop eem-policy.zip which contains connection settings to the Event Gateway.<br>

![](./images/images/acedb-config-kafka-policy.png)

<br>
Click on "Create Configuration" <br>
Select "Trust Store", drag & drop the bootstrap.jks that you created in lab1. The bootstap.jks contains certificate of the Event Gateway.<br>


![](./images/images/acedb-config-eem-truststore-jks.png)

Click on "Create Configuration" <br>
Select "setdbparms.txt"<br>
Copy & paste the below configuration lines.<br>
![](./images/images/acedb-config-eem-setdbparms.png)
<br>
kafka::eem-security-identity \<apic-dev-portal-app-key-from-lab1\> \<apic-dev-portal-app-secret-from-lab1\>  <br>
truststore::eem-truststore-password dummy passw0rd

<br>


### Deploying Bar File
Navigate to the App Connect Dashboard Home page, click on "Deploy Integrations".<br>
Select "Quick Start Integration", click "Next".<br>

![](./images/images/acedb-deploy-1.png)

Keep Toolkit Window on the left, and drag & drop the bar file that you built as in below.<br>
![](./images/images/acedb-deploy-2.png)

<br>
Click "Next" & search for "eem" Configuration.<br>
Select the 3 configurations that you have created above.<br>

![](./images/images/acedb-deploy-3.png)
Click "Next"<br>
Set the "Name" as ace-tk-eem-flight-landing-events.<br>
Click "Create" on the top right.<br>

<br>
This should create the Integration Runtime with the flow that you have built.<br>
<br>

## 3. Results

Logon to the OpenShift Console, and select the App Connect Project (namespace).<br>
Navigate to Workloads > Pods > search for ace-tk-eem-flight-landing-events pod.<br>
Click on "Terminal" Tab <br>
```
cd /tmp
ls -l student*
tail -f studentxx_flight_landings.txt
```
if you have completed lab2, then the events should be publishing to the Kafka Topic STUDENTXX.FLIGHT.LANDINGS.<br>
The events should be appended to studentxx_flight_landings.txt as below <br>
![](./images/images/ocp-results-1.png)

<br>
!!! CONGRATULATIONS !!!