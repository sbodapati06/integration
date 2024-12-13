# API Connect - Generate Testcases Automatically using the SmartGenerator 

# 1. Overview

In this lab, you will explore "Test APIs" feature of IBM API Connect that can automatically generate test cases to execute calls and trace the API actions. By using IBM Api Connect TestAPIs capability you can deliver high quality APIs consistently and improve developer productivity with automation and AI powered testing capabilities.
<br>
Reference: https://www.ibm.com/products/api-connect/api-testing

You will be using Customer Database Rest API deployed to explore Test APIs capability of IBM API Connect. <br>
<br>


## 2. Api Connect - Test APIs

From the Cloud Pak for Integration Platform Navigator, open API Connect Manager (apim-demo). <br>

![alt text](./images/image-9.png)

Click on the "Test APIs" tile. <br>

![alt text](./images/image.png)

## 3. Add Test Suite

![alt text](./images/image-1.png)


Click on \"Add test suite\". <br>


![alt text](./images/image-2.png)


![alt text](./images/image-3.png)

Click on the ts-customer-database test suite. <br>

![alt text](./images/image-4.png)

## 3. Add Test

Click \<Add\> to add a test. <br>

![alt text](./images/image-5.png)
\<Next\>

![alt text](./images/image-6.png)

Select the Customer Database REST API you deployed in the API Management first lab. <br>
\<Next\>

![alt text](./images/image-7.png)

![alt text](./images/image-8.png)

Hit \<Save\> button on the top right of the screen. <br>

<br>


## 4. Set Global Datasets

Now, lets update the "Global Datasets". <br>

![alt text](./images/image-10.png)

Before we update the "Global Datasets" lets capture the Customer Database API's API Gateway URL from the Developer Portal. <br>

Click on the Home icon on the top left. <br>

Click on "Manage Catalogs" tile. <br>
![alt text](./images/image-11.png)

Click on "Sandbox" catalog tile. <br>

Click on "Catalog Settings" tab, and Portal on the left. <br>

![alt text](./images/image-12.png)

Once the Developer Portal is open then click on the Product that the Customer Database API is published. <br>

Select the Customer Database API, and capture the domain as below. <br>

![alt text](./images/image-13.png)

Copy that select section and paste into a Notepad. <br>

Now, lets go back to the "Test APIs" and open the Test you created previously. <br>


Click on the "Data Sets" section. <br>

Edit domain from "yourdomain.com" with what you captured in the Developer Portal. <br>

![alt text](./images/image-14.png)

Edit the basePath and set to "customerdb/v1". <br>

![alt text](./images/image-15.png)


Hit the \<Save\> button on the top right. <br>




## 5. Set Input Datasets

![alt text](./images/image-19.png)

### 5a. Update X-IBM-Client-Id, and X-IBM-Client-Secret 

In the API Connect Developer Portal lab, you must have created an application and obtained ClientId, and ClientSecret. Lets populate those values as below. <br>

![alt text](./images/image-20.png)

<br><br>


## 5. Disable Security

Disable security to the API Gateway URL by adding insecureSkipVerify flag to each operation.<br>
```
      insecureSkipVerify: true
```

![alt text](./images/image-16.png)


<br>

## 6. Mockup Customer Record

Update POST operation, enter mockup data as below. <br>

![alt text](./images/image-18.png)

Hit \<Save\> button on the top right. <br>




## 7. Replace customerId parm with id

The POST operation returns and "id" back so we should change customers.customerId with customers.id in the GET and DELETE operations. <br>

![alt text](./images/image-17.png)

Hit \<Save\> button on the top right. <br>


## 8. Add body element to DELETE operation

The Customer Database API's DELETE operation is expecting a non null body. Let's append body: with curly open&close curly brackets. <br>

![alt text](./images/image-23.png)

## 8. Update Authorization header element of DELETE operation

```
Authorization: secr3t
```
![alt text](./images/image-24.png)

<br>



## 8. Run Test

Hit "Run Test" button. <br>

![alt text](./images/image-21.png)

![alt text](./images/image-22.png)

![alt text](./images/image-25.png)


## 8. Optional - Running the Test from Command line

in this section we will create a webhook, and apikey to run the test from a command line.
<br>

Click on your Test Suite. <br>

![alt text](./images/image-26.png)


### 8a. Publish the Test
![alt text](./images/image-30.png)

Click \<Yes\>. <br>

![alt text](./images/image-31.png)



### 8b. Create webhook

Click on the Settings tab. <br>

![alt text](./images/image-27.png)

Click on "API hooks and keys" and Click on \<Add\> button in the Hooks section. <br>
![alt text](./images/image-28.png)

Copy and save the URL to Notepad. <br>
![alt text](./images/image-29.png)

### 8b. Create key

![alt text](./images/image-32.png)

Click \<Add\>. <br>

![alt text](./images/image-33.png)

Capture the Key, Secret into the Notepad.


### 8c. Run from the Command line

```
export KEY=PASTE-THE-KEY
export SECRET=PASTE-THE-SECRET
export WEBHOOK=PASTE-THE-WEBHOOK-URL
curl -k -X POST  -H X-API-Key:$KEY -H X-API-Secret:$SECRET  ${WEBHOOK}/tests/run" -d ' { "options": { "allAssertions": true }  } ' | jq
```

You should see output like below with 200 status codes. <br>

![alt text](./images/image-34.png)

<br><br><br><br>


## 9. Reference

### 9a. Completed Testcase

```
info:
    testName: 1-Customer-Database-Rest-Api
    version: 2
configs:
    globalVariables:
        basePath: customerdb/v1
        domain: apim-demo-gw-gateway-cp4i-apic.apps.67448d46e8a5a32b1ae1a48a.ocp.techzone.ibm.com/sbodapati-porg/sandbox
        protocol: https://
    inputs:
        - default:
            X-IBM-Client-Id: a441bbb44dd8687b78cc1be9da33ce0c
            X-IBM-Client-Secret: d0e6b28b232f705c868f255daf2822a8
            customerId: ""
            token: token
steps:
    - type: request
      method: post
      url: '{{ protocol }}{{ domain }}/{{ basePath }}/customers'
      var: customers
      body:
        address: 175 Marina Drive, Edison, NJ, 08824
        firstname: Joseph
        lastname: Jodl
      mode: json
      headers:
        Authorization: Bearer {{ token }}
        Content-Type: application/json
        X-IBM-Client-Id: '{{ X-IBM-Client-Id }}'
        X-IBM-Client-Secret: '{{ X-IBM-Client-Secret }}'
      insecureSkipVerify: true
    - type: request
      method: get
      url: '{{ protocol }}{{ domain }}/{{ basePath }}/customers/{{ customers.id }}'
      var: payload
      mode: json
      headers:
        Authorization: Bearer {{ token }}
        X-IBM-Client-Id: '{{ X-IBM-Client-Id }}'
        X-IBM-Client-Secret: '{{ X-IBM-Client-Secret }}'
      insecureSkipVerify: true
    - type: assert-equals
      expression: payload_response_statusCode
      value: "200"
      stoponfail: "false"
    - type: assert-exists
      expression: payload.address
      stoponfail: "true"
    - type: assert-is
      expression: payload.address
      value: string
    - type: assert-exists
      expression: payload.firstname
      stoponfail: "true"
    - type: assert-is
      expression: payload.firstname
      value: string
    - type: assert-is
      expression: payload.id
      value: number
    - type: assert-exists
      expression: payload.lastname
      stoponfail: "true"
    - type: assert-is
      expression: payload.lastname
      value: string
    - type: request
      method: delete
      url: '{{ protocol }}{{ domain }}/{{ basePath }}/customers/{{ customers.id }}'
      body: {}
      mode: json
      headers:
        Authorization: secr3t
        X-IBM-Client-Id: '{{ X-IBM-Client-Id }}'
        X-IBM-Client-Secret: '{{ X-IBM-Client-Secret }}'
      insecureSkipVerify: true

```

### Congratulations !!!