# API Connect Test APIs - Generate Tests using SmartGenerator

# 1. Overview

In this lab, you will explore "Test APIs" capability of IBM API Connect to generate tests automatically. By using IBM Api Connect TestAPIs capability you can deliver high quality APIs consistently and improve developer productivity with automation and AI powered testing capabilities.
<br>
Reference: https://www.ibm.com/products/api-connect/api-testing

You will be using Customer Database Rest API deployed in lab xyz to explore Test APIs capability of IBM API Connect. <br>
<br>

# 2. App Connect - Download Swagger document

From the Cloud Pak for Integration Platform Navigator, open App Connect Dashboard (ace-dashboard). <br>

Click on the Runtimes.<br>
Select customerdatabasev1 Integration Runtime, and click on the API, then "Download the OpenAPI Document" as below.<br>
![Alt text](./images/cdb-openapi.png)

If you haven't deployed the customerdatabasev1 integration runtime, then download the bar file from [<b><u>here</u></b>](./src/CustomerDatabaseV1.bar), and deploy into App Connect Dashboard.<b>
(OR)
Download the yaml from here.<br>
[<b><u>Customer_database_1.0.0.yaml</u></b>](./src/Customer_database_1.0.0.yaml)

# 3. Api Connect - Test APIs

From the Cloud Pak for Integration Platform Navigator, open API Connect Manager (apim-demo). <br>

Click on the "Test APIs" tile. <br>

![Alt text](./images/image.png)

![Alt text](./images/image-2.png)
<br>
Click on "Create Test Suite". <br>

![Alt text](./images/image-3.png)

Click correct tick mark (on top right).<br>

![Alt text](./images/image-4.png)

Click on "Tests" Option.<br>

![Alt text](./images/image-5.png)
<br>
Click on \<Add\> to add a test. <br>

![Alt text](./images/image-6.png)
<br>
Enter customerdatabasev1_test1 and hit continue.<br>

![Alt text](./images/image-7.png)
Click \<Template from Specifications\> in the bottom middle option.<br>


Select "Swagger 2.0 YAML", and click on "drag a file here or click to upload", select downloaded yaml file (as below). <br>

![Alt text](./images/image-19.png)


Click on the dropdown, and select /customerdb/v1/customers/{customerId} operation.<br>
Select "Smart Generator" circle as below, and click on the Tick mark on right of the window.<br>
![Alt text](./images/image-20.png)

That should generate a POST, GET, and DELETE operations as below.<br>
![Alt text](./images/image-21.png)

Now, add a variable "customerId" in between POST, and GET as below.<br>

Click on the (i) circle on the GET opeartion, and select "add component before".<br>

![Alt text](./images/image-22.png)

Click on "Set variable" circle.<br>

![Alt text](./images/image-23.png)

Set the variable name as customerId, and value as ${payload.id}. <br>

Confirm changes by clicking the Tick mark on top right of the Variable window.<br>

![Alt text](./images/variable.png)

Click \<Save\> button on the top, and then click \<Run Test\>. <br>

![Alt text](./images/image-24.png)

![Alt text](./images/image-25.png)

Take the defaults, and hit \<Run Test\>, and that should generate a report as below.<br>

![Alt text](./images/image-26.png)

Review and Close the Report.<br>

Click on \<Save and Exit\> button on the top.<br>

![Alt text](./images/image-27.png)

![Alt text](./images/image-28.png)


NOW CLICK \<Publish\> button as below.<br>
![Alt text](./images/image-29.png)

Click on the Tests option on the top.<br>
![Alt text](./images/image-31.png)

Click on the 3 dots on the test CustomerDatabasev1_test1 as below, and click Run. You can hit Run few times.<br>
![Alt text](./images/image-32.png)

Click on the Dashboard on the top.<br>
![Alt text](./images/image-34.png)

Here you can see the tests that were ran, and results with Graphs.<br>
![Alt text](./images/image-33.png)



### Congratulations !!!

