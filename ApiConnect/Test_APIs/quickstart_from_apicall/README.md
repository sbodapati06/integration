# API Connect - Test API's

# 1. Overview

In this lab, you will explore "Test APIs" capability of IBM API Connect to generate tests automatically. By using IBM Api Connect TestAPIs capability you can deliver high quality APIs consistently and improve developer productivity with automation and AI powered testing capabilities.
<br>
Reference: https://www.ibm.com/products/api-connect/api-testing

<br>
You will be using Customer Database Rest API deployed in lab xyz to explore Test APIs capability of IBM API Connect. <br>

<br>

# 2. App Connect - Deploy Customer Database Rest API

lab xxx to deploy the customer database bar file into App Connect.


# 3. Api Connect - Test APIs

From the Cloud Pak for Integration Platform Navigator, open API Connect Manager (apim-demo). <br>

Click on the "Test APIs" tile. <br>

![Alt text](./images/image.png)

![Alt text](./images/image-2.png)
Click on "create a test suite". <br>

![Alt text](./images/image-3.png)

Click correct tick mark (on top right).<br>

![Alt text](./images/image-4.png)

Click on "Tests" Option.<br>

![Alt text](./images/image-5.png)
Click on \<Add\> to add a test. <br>

![Alt text](./images/image-6.png)
Enter customerdatabasev1_test1 and hit continue.<br>

![Alt text](./images/image-7.png)
Click \<Edit\> in the bottom left.<br>


![Alt text](./images/image-1.png)
Select the first tile "Quick start from an API Call". <br>

![Alt text](./images/image-8.png)

Open App Connect Dashboard, and copy the URL for postCustomers method as below.<br>
![Alt text](./images/image-9.png)

Switch back to API Connect Test API's, select POST method, paste the URL as below.<br>

![Alt text](./images/image-10.png)

'''
In the body, set Raw & "application/json".
{
  "firstname": "Victor",
  "lastname": "Kok",
  "address": "1675 Laeh Park"
}
'''
Try the request by hitting \<Send\>. <br>

![Alt text](./images/image-11.png)

Hit \<Generate Test\>, this will generate the testcase, and assertions.<br>

![Alt text](./images/image-12.png)

Click \<close\>. <br>

That should a testcase, and assertions for you. <br>

![Alt text](./images/image-13.png)

Lets add more testcases getcustomerbyid, deletecustomerbyid operations.<br>

Hit the plus sign on the top left as in below.<br>
![Alt text](./images/image-14.png)

Add a Variable first.<br>
![Alt text](./images/image-15.png)

Name the variable as customer_id, and value to be ${payload.id}.<br>

![Alt text](./images/image-17.png)


Hit the plus sign on the top left as in below.<br>
![Alt text](./images/image-14.png)
Select GET operation. <br>

Back to App Connect Dashboard, and capture URL for "GET /customers/{customerId}".<br>
![Alt text](./images/image-16.png)



Switch back to API Connect Test API's, and paste the URL.
![Alt text](./images/image-18.png)



## TO BE CONTINUE