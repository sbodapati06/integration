# APIC GraphQL Installation on openshift

---

# Table of Contents
- [1. Install postgresql database](#postgresql-install)
- [2. Create Secrets](#secrets)
- [3. Download APIC GraphQL Bundle](#download)
- [4. Setup routes](#routes)
- [5. Verify installation](#verify)
- [6. Add User Accounts](#users)
- [7. Deploy & Test Sample GraphQL API](#deploy-sample-graphql-api)
---


## 1. Install postgresql database <a name="postgresql-install"></a>

create postgresql namespace
```
oc new-project postgresql
```

The script will apply the below yaml’s into postgresql namespace.
```
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  labels:
    operators.coreos.com/crunchy-postgres-operator.openshift-operators: ""
  name: postgresql
  namespace: postgresql
spec:
  channel: v5
  installPlanApproval: Automatic
  name: crunchy-postgres-operator
  source: certified-operators
  sourceNamespace: openshift-marketplace
  startingCSV: postgresoperator.v5.7.1
```

```
old - do not use...
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  labels:
    operators.coreos.com/postgresql.openshift-operators: ''
  name: postgresql
  namespace: openshift-operators
spec:
  channel: v5
  installPlanApproval: Automatic
  name: postgresql
  source: community-operators
  sourceNamespace: openshift-marketplace
```

Note: ignore the dummy tables, they are not used by GraphQL.<br>
```
kind: ConfigMap
apiVersion: v1
metadata:
    name: pg-apic-graphql-data-cm
    namespace: postgresql
data:
    demo_data.sql: |+
        \c pg-apic-graphql
        ALTER DEFAULT PRIVILEGES GRANT ALL ON TABLES TO pgadmin;

        set schema 'public';

        CREATE TABLE buildings (buildingid VARCHAR(5) NOT NULL, streetname VARCHAR(100) NOT NULL, securitycontact VARCHAR(30) NOT NULL, PRIMARY KEY (buildingid) );

        INSERT INTO buildings VALUES ( 'A',  'Warren Spinney',     '+1 202-918-2132');
        INSERT INTO buildings VALUES ( 'B',  'Aigburth Hall Road', '+1 202-918-3488');
        INSERT INTO buildings VALUES ( 'DM', 'Hinton Leaze',       '+1 202-918-2248');
        INSERT INTO buildings VALUES ( 'DE', 'Westgate Mill',      '+1 202-918-6611');
        INSERT INTO buildings VALUES ( 'G',  'Heartswood Road',    '+1 202-918-7019');
        INSERT INTO buildings VALUES ( 'H',  'Denbigh Crest',      '+1 202-918-1911');

        CREATE TABLE warehouses (warehouseid VARCHAR(7) NOT NULL, location VARCHAR(20) NOT NULL, sqft INTEGER NOT NULL, managerid VARCHAR(10), status INTEGER, PRIMARY KEY (warehouseid) );

        INSERT INTO warehouses VALUES ('North',   'Beyghworth', 25000,  '86621106601', 0);
        INSERT INTO warehouses VALUES ('South',   'New Kenroy', 82000,  '13303921130', 0);
        INSERT INTO warehouses VALUES ('West',    'West Vil',   110000, '11031941444', 0);
        INSERT INTO warehouses VALUES ('East',    'Tunheb',     33500,  '13910006610', 1);
        INSERT INTO warehouses VALUES ('Central', 'Leicamport', 41000,  '93111071832', 0);
```

```
kind: PostgresCluster
metadata:
  name: pg-apic-graphql
  namespace: postgresql
spec:
  backups:
    pgbackrest:
      repos:
      - name: repo1
        volume:
          volumeClaimSpec:
            accessModes:
            - ReadWriteOnce
            resources:
              requests:
                storage: 1Gi
  databaseInitSQL:
    name: pg-apic-graphql-data-cm
    key: demo_data.sql
  instances:
  - dataVolumeClaimSpec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
    name: ""
    replicas: 1
  openshift: true
  patroni:
    leaderLeaseDurationSeconds: 30
    port: 8008
    syncPeriodSeconds: 10
    dynamicConfiguration:
      postgresql:
        pg_hba:
          - hostnossl pg-apic-graphql demouser all md5
          - hostssl pg-apic-graphql demouser all scram-sha-256
  port: 5432
  postgresVersion: 15
  proxy:
    pgBouncer:
      port: 5432
      replicas: 1
  users:
  - name: demouser
    databases:
      - pg-apic-graphql
    options: SUPERUSER
  - name: pgadmin
    databases:
      - pg-apic-graphql
```



## 2. Create Secrets <a name="secrets"></a>

```
oc new-project apic-grahql


# ibm-entitlement-key
oc create secret docker-registry ibm-entitlement-key --docker-server=cp.icr.io --docker-username=cp --docker-password=xxx.xxx.xxxxx


# postgresql db-secret
oc create secret generic db-secret --from-literal=DSN=postgresql://demouser:6Rk%2BW0U-%2CK4pyA3v%3D%5De%7Cd.33@pg-apic-graphql-primary.postgresql.svc:5432/pg-apic-graphql

where <dsn> is a string of the form postgresql://user:password@host/database.
where, demouser is postgresql secret ending with demouser.

``` 
![alt text](./images/image-1.png)


## 3. Download APIC GraphQL Bundle <a name="download-apic-biunary"></a>

Download APIC GraphQL installation package from here. <br>
https://github.com/IBM/cloud-pak/tree/master/repo/case/ibm-stepzen-case

```
tar -xvf *.tgz

cd /Users/sbodapati/xibm_ts/sb_demos/apiconnect/apic-graphql/ibm-stepzen-case/inventory/stepzenGraphOperator/files/deploy

oc apply -f crd.yaml

sed -e "s/namespace: stepzen/namespace: apic-graphql}/" \
  inventory/stepzenGraphOperator/files/deploy/operator.yaml > .operator-temp.yaml

oc apply -f operator-temp.yaml

update cr.yaml, add below field to the spec.
spec: 
  enableLocalhost: true

oc apply -f cr.yaml


oc get StepzenGraphServer
NAME      STATUS   SUMMARY              AGE
stepzen   Ready    Services are ready   4h14m
```


## 4. Setup routes <a name="routes"></a>

NOTE: REPLACE THE DNS on host: field. <br>

```
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  annotations:
    cert-manager.io/issuer-kind: ClusterIssuer
    cert-manager.io/issuer-name: letsencrypt
    haproxy.router.openshift.io/balance: random
    haproxy.router.openshift.io/disable_cookies: "true"
    haproxy.router.openshift.io/hsts_header: max-age=31536000;includeSubDomains;preload
    haproxy.router.openshift.io/timeout: 30s
    haproxy.router.openshift.io/timeout-tunnel: 5d
  name: stepzen-to-graph-server
spec:
  host: stepzen.apps.66c8d24ab5e59d371dc0dea5.ocp.techzone.ibm.com
  port:
    targetPort: stepzen-graph-server
  tls:
    haproxy.router.openshift.io/hsts_header: max-age=31536000;includeSubDomains;preload
    insecureEdgeTerminationPolicy: None
    termination: edge
  to:
    kind: Service
    name: stepzen-graph-server
    weight: 200
  wildcardPolicy: None
```

```
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  annotations:
    cert-manager.io/issuer-kind: ClusterIssuer
    cert-manager.io/issuer-name: letsencrypt
    haproxy.router.openshift.io/balance: random
    haproxy.router.openshift.io/disable_cookies: "true"
    haproxy.router.openshift.io/hsts_header: max-age=31536000;includeSubDomains;preload
    haproxy.router.openshift.io/timeout: 30s
    haproxy.router.openshift.io/timeout-tunnel: 5d
  name: graphql-to-graph-server
spec:
  host: graphql.apps.66c8d24ab5e59d371dc0dea5.ocp.techzone.ibm.com
  path: /
  port:
    targetPort: stepzen-graph-server
  tls:
    insecureEdgeTerminationPolicy: None
    termination: edge
  to:
    kind: Service
    name: stepzen-graph-server
    weight: 150
  wildcardPolicy: None
```

```
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  annotations:
    cert-manager.io/issuer-kind: ClusterIssuer
    cert-manager.io/issuer-name: letsencrypt
    haproxy.router.openshift.io/balance: random
    haproxy.router.openshift.io/disable_cookies: "true"
    haproxy.router.openshift.io/hsts_header: max-age=31536000;includeSubDomains;preload
    haproxy.router.openshift.io/timeout: 30s
    haproxy.router.openshift.io/timeout-tunnel: 5d
  name: graphql-to-graph-server-subscriptions
spec:
  host: graphql.apps.66c8d24ab5e59d371dc0dea5.ocp.techzone.ibm.com
  path: /stepzen-subscriptions/
  port:
    targetPort: stepzen-graph-server-subscription
  tls:
    insecureEdgeTerminationPolicy: None
    termination: edge
  to:
    kind: Service
    name: stepzen-graph-server-subscription
    weight: 150
  wildcardPolicy: None
```

## 5. Verify installation <a name="verify"></a>
 
![alt text](./images/image.png)

Grab the link, and append /version
https://stepzen.apps.66c8d24ab5e59d371dc0dea5.cloud.techzone.ibm.com/version
{
   "Version": "Production",
   "Service": "ZenServ",
   "GitSha": "8654b813",
   "DeploymentType": "local",
   "Runtime": "go1.21.9"
}

## 6. Add User Accounts <a name="users"></a>

### Username: graphql

```
oc project apic-graphql

cd /Users/sbodapati/xibm_ts/sb_demos/apiconnect/apic-graphql/ibm-stepzen-case/inventory/stepzenGraphOperator/files/deploy

./stepzen-admin.sh add-account graphql


Output
./stepzen-admin.sh add-account graphql
[/] Waiting for pod to complete... stepzen-admin-jobs-9aab1f-2nsr7
account: graphql
adminkey: graphql::local.io+1000::b51d54dea10ccc3991d86f68c4d22f3cf3a3795e7b8c15f8b1826c72c8ab9997
apikey: graphql::local.net+1000::a56b6865f2a17031ba8f72524d16fbb1c2c037aa96eb29b54e595a2ea423604b

# stepzen login using adminkey
stepzen login -a graphql -k graphql::local.io+1000::b51d54dea10ccc3991d86f68c4d22f3cf3a3795e7b8c15f8b1826c72c8ab9997 apps.67448d46e8a5a32b1ae1a48a.ocp.techzone.ibm.com
```

Note: use “npm install -g stepzen”  if not installed already. <br>


## 7. Deploy & Test Sample graphql API <a name="deploy-sample-graphql-api"></a>

Create a sample graphql API. <br>

```
stepzen init --endpoint=api/product-demo

stepzen import curl "https://introspection.apis.stepzen.com/customers" --query-name "customers"
```

Note: Disable security – run the below command before “stepzen start”. <br
This will create config.yaml.<br>
```
echo "access:
  policies:
    - type: Query
      policyDefault:
        condition: true" > config.yaml
```

```
stepzen start
```

![alt text](./images/image-2.png)

Access the local test environment.<br> 

http://localhost:5001/api/product-demo

<br> 

![alt text](./images/image-3.png)


#### End ######
