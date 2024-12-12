
## 1. Sender / Receiver Connectivity - OnPrem to CP4I Queue Manager

In this blog, we will establish OnPrem to Cp4I sender/receiver connectivity.
<br>

### 1.1 Queue Manager configuration - CP4I Side

CP4I queue manager name is nativeha. <br>

1) Create Receiver channel <br>
2) Create SNI Route <br>

```
runmqsc nativeha
-- define receiver channel
def chl(UNIRDQM1.NATIVEHA) chltype(rcvr) sslciph(ANY_TLS12_OR_HIGHER) sslcauth(optional)
```

Create a Route like below. Notice that host: is using SNI naming convension.
1) the host should be lowercase <br>
2) Our channel name is UNIRDQM1.NATIVEHA, and it has a dot (.) in the channel name. The dot should be replaced with 2e-
Example: unirdqm1.nativeha should be set as unirdqm12e-nativeha <br>
3) Run the below yaml from either OCP Console, or Command Line using oc apply -f <br>
```
kind: Route
apiVersion: route.openshift.io/v1
metadata:
  name: nativeha-mq-traffic-unirdqm1-nativeha
  namespace: cp4i-mq
spec:
  host: unirdqm12e-nativeha.chl.mq.ibm.com
  to:
    kind: Service
    name: nativeha-ibm-mq
  port:
    targetPort: 1414
  tls:
    termination: passthrough
  wildcardPolicy: None
```


### 1.2 Queue Manager Configuration - OnPrem side 

OnPrem Queue Manager name is UNIRDQM1. <br>

1) Extract CP4I Queue Manger Certificate <br>
2) Add the certificate to key.kdb
2) Define Sender Channel from ONPREM queue manager to CP4I Queue Manager (UNIRDQM1.NATIVEHA) <br>

```
cd /var/mqm/qmgrs/<qmgrname>/ssl

# Extract nativeha queue manager certificate
oc project cp4i-mq

# oc get route "<Queue Manager Name>-ibm-mq-qm")> -o jsonpath="{.spec.host}"
Reference: https://www.ibm.com/docs/en/ibm-mq/9.3?topic=cqmumo-configuring-route-connect-queue-manager-from-outside-red-hat-openshift-cluster
export QMGR_HOST=`oc get route nativeha-ibm-mq-qm -o jsonpath="{.spec.host}"`
  Output:
  nativeha-ibm-mq-qm-cp4i-mq.ocp-dev-xxxx-0000.us-east.containers.appdomain.cloud

openssl s_client -connect $QMGR_HOST:443 -servername $QMGR_HOST -showcerts | openssl x509 > nativeha-qmgr.crt

if there isn't a key.kdb file then run the below command, otherwise run next command to add the nativeha queue manager certificate.
runmqakm -keydb -create -db key.kdb -type cms -pw passw0rd -stash

runmqakm -cert -add -db key.kdb -pw passw0rd -label nativeha -file nativeha-qmgr.crt
```

```
runmqsc UNIRDQM1 

def chl(UNIRDQM1.NATIVEHA) chltype(sdr) conname('nativeha-ibm-mq-qm-cp4i-mq.ocp-dev-xxxx-0000.us-east.containers.appdomain.cloud(443)') sslciph(ANY_TLS12_OR_HIGHER) sslcauth(optional) xmitq('nativeha')

def qlocal('nativeha') usage(xmitq) 

refresh security (*)

START CHANNEL(UNIRDQM1.NATIVEHA)

dis chstatus(unirdqm1.nativeha)
     1 : dis chstatus(unirdqm1.nativeha)
AMQ8417I: Display Channel Status details.
   CHANNEL(UNIRDQM1.NATIVEHA)              CHLTYPE(SDR)
   CONNAME(169.63.130.26(443))             CURRENT
   RQMNAME(nativeha)                       STATUS(RUNNING)
   SUBSTATE(MQGET)                         XMITQ(nativeha)
```

Note: Check queue manager logs on both sides if any issues with starting the channel.


