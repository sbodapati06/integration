# Api Connect & API Gateway upgrades



## 1. Upgrading Api Gateway from v10.5.0.2 to v10.5.0.3

Environment: <br>
Openshift 4.12 <br>
Api Connect : v10.0.5.2-ifix1

```
$ oc edit gatewaycluster \<name-of-the-apigateway>
```
Add the below template section:
```
  license:
    accept: true
    license: L-GVEN-GFUPVE
    metric: VIRTUAL_PROCESSOR_CORE
    use: nonproduction

  template:
  - name: datapower
    containers:
    - name: gateway
      image: cp.icr.io/cp/datapower/datapower-cp4i:10.5.0.3
```

Notes: <br>
1. Updating DataPowerService yaml & the version from 10.5.0.2 to 10.5.0.3 isn't working. The gateway pod going into crashloopbackoff. Hence the workaround above.<br>
2. The template workaround above (updating the GatewayCluster), will upgrade DataPower to 10.5.0.3, but In OpenShift console it still shows 10.5.0.2 for the GatewayService instance. THIS IS EXPECTED! 
<br>