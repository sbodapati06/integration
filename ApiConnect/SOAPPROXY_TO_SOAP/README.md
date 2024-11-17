# Creating SOAP Proxy based upon a WSDL

# 1. Overview

In this lab, you will expose a SOAP WebService as SOAP Proxy in API Connect. You will be deploying a very simple Temperature Converter WebService.  The Temperature Converter WebService is built on App Connect Enterprise Toolkit, and the bar file will be deployed to App Connect Dashboard on Cloud Pak for Integration.<br>

<b> Design diagram </b>
<br>
![Alt text](./images/design-diagram.png)
<br>

# 2. App Connect - Deploy Temperature Converter WebService

<b>Download the bar file from [<b><u>HERE</u></b>](./src/TemperatureConverter.bar).
</b>

Deploy the bar file.<br>

Logon to Cloud Pak for Integration Platform Navigator, open App Connect Dashboard > Click on "Deploy Integrations" tile.<br>

![App Connect Dashboard](./images/image.png)

Select "Quick Start Integration", and Click \<Next\>. <br>

Drag & drop the bar file downloaded above as below.<br>
![Alt text](./images/image-1.png)

Click \<Next\> two times. <br>

Name your Integration Runtime as \"ace-tk-temperature-converter\". <br>

![Alt text](./images/image-3.png)

Click \<Create\>. <br>

Wait for 30seconds and refresh the page. <br>

Make sure the Integration Runtime \"ace-tk-temperature-converter\" is Ready. <br>

Click on the Integration Runtime \"ace-tk-temperature-converter\" tile.<br>

![Alt text](./images/image-13.png)

![Alt text](./images/image-14.png)

Click on the Properties tab, and copy "SOAP HTTP URL". This will be configured in the next section (API Creation).<br>


![Alt text](./images/image-12.png)




# 3. Api Connect - Create SOAP Proxy from the WSDL

Download the zip file that contains Temperature Converter WSDL, and XSD's from [<b><u>here</u></b>](./src/TemperatureConverter_WSDL.zip).

Logon to Cloud Pak for Integration, and open API Management (apim-demo). <br>

Select "Develop APIs and Products" tile.<br>

![Alt text](./images/image-2.png)

Click on "Add" > API button on the top right of the screen.<br>

![Alt text](./images/image-4.png)

Select the API Type "From Existing WSDL Service (SOAP Proxy). <br>

Click \<Next\> in the bottom right of of the screen. <br>

Drag and drop the zip file downloaded above.<br>
![Alt text](./images/image-5.png)

Click \<Next\> in the bottom right of of the screen. <br>

![Alt text](./images/image-6.png)

Click \<Next\> in the bottom right of of the screen. <br>


Change the API name as studentxx-temperature-converter-soap-api
<br>
![Alt text](./images/image-7.png)

Click \<Next\> in the bottom right of of the screen. <br>

![Alt text](./images/image-8.png)

Click \<Next\> in the bottom right of of the screen. <br>


![Alt text](./images/image-9.png)

Click \<Edit API\> in the bottom right of of the screen. <br>

![alt text](./images/image-9a.png)

Click on "Host", and blank out the value. <br>
![Alt text](./images/image-11.png)

Click on "Gateway" Tab. <br>

![Alt text](./images/image-10.png)

Close the Node pallete.<br>

<br>
Now, complete the API design. <br><br>

Click on "Properties" on the left, and click (+) sign. Add target-url property, and paste "SOAP HTTP URL" captured in the previous section as below. <br>

![Alt text](./images/image-15.png)

Click \<Create\>. <br>

Now, click on "Policies" option on the left, and lets modify the API in the designer view.<br>

<br>
Now click on the  "Invoke" Node, and update the URL. <br>

![Alt text](./images/image-16.png)

Update URL value to "{target-url}" (without the double quotes). <br>

SAVE the API (The Save button is on the top right of the screen). <br>

<br>

# 4. Testing the SOAP Proxy

Click on the "Test" tab.<br>
![alt text](./images/image-17.png)

Click on "Test Configuration". <br>
![alt text](./images/image-17a.png)

Enable Auto-publish, and click "Save Preferences".<br>
![Alt text](./images/image-18.png)

Now, the API should be online.<br>
![alt text](./images/image-18a.png)

Select POST and CtoF operation. <br>

![Alt text](./images/image-20.png)

Enter sample SOAP input payload in the Body Tab, and hit the <Send> button.<br>

```
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:tem="http://TemperatureConverter">
   <soapenv:Body>
      <tem:CtoF>
         <TemperatureInC>0</TemperatureInC>
      </tem:CtoF>
   </soapenv:Body>
</soapenv:Envelope>
```

You should get the response with the Converted Fahrenheit value as below.<br>
![Alt text](./images/image-19.png)

Check the "trace", and you should see as below.<br>
![Alt text](./images/image-21.png)


Similary, test FtoC method.<br>
```
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:tem="http://TemperatureConverter">
   <soapenv:Body>
      <tem:FtoC>
         <TemperatureInF>32</TemperatureInF>
      </tem:FtoC>
   </soapenv:Body>
</soapenv:Envelope>
```
![alt text](./images/image-22.png)

You should get a response as below, converted temperature from Farenheit to Centigrade.<br>

![alt text](./images/image-23.png)

### Congratulations!!!





