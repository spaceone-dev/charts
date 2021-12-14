# Kubernetes namespace

~~~
kubectl create ns spaceone
kubectl create ns root-supervisor

alias kcd='kubectl config set-context $(kubectl config current-context) --namespace'
kcd spaceone
~~~

# Helm repo

~~~
helm repo add spaceone https://spaceone-dev.github.io/charts
helm repo update
helm search repo
~~~

# Install

## Type 1. (for developer)
* update values.yaml
* update frontend.yaml
* MongoDB is a temporary POD deployment.

** frontend.yaml **
To exact configuration, update frontend.yaml file.

For the HTTPS connection, you have to prepare certificate a.k.a. AWS Certificate ARN.

If your domain is ***example.com***, you have to create certificate for ****.console.example.com*** and ***console-api.example.com***.


You have to update your domain settings.

| Component |	Key 				| Description |
| --- 		| --- 				| --- |
| console	| ENDPOINT 			| ENDPOINT of console-api FQDN |
| console	| host				| FQDN of User domains |
| console	| alb.ingress.kubernetes.io/certificate-arn |  Certificate ARN |
| console 	| external-dns.alpha.kubernetes.io/hostname | External DNS name of console	|
| console-api	| host				| Hostname of console-api |
| console-api	| alb.ingress.kubernetes.io/certificate-arn |  Certificate ARN |
| console-api	| external-dns.alpha.kubernetes.io/hostname | External DNS name of console-api	|

~~~
kcd spaceone

helm install spaceone -f values.yaml -f frontend.yaml spaceone/spaceone

~~~


## Type 2. (for production)

For production level, install mongodb cluster or AWS document DB

If you use mongodb cluster,
host is "localhost" in database.yaml
Use TYPE 2. global varable in values.yaml

~~~
kcd spaceone

helm install spaceone -f values.yaml -f frontend.yaml -f database.yaml spaceone/spaceone

~~~


# Upgrade
## Changed Configuration
### image versions
- 1.8.7

### value.yaml
- [ADD] New service - cost-analysis
```diff
+cost-analysis:
+    enabled: true
+    scheduler: false
+    worker: true
+    replicas: 1
+    replicas_worker: 2
+    image:
+      name: public.ecr.aws/megazone/spaceone/cost-analysis
+      version: 1.8.7
+
+    application_grpc:
+        DEFAULT_EXCHANGE_RATE:
+            KRW: 1178.7
+            JPY: 114.2
+            CNY: 6.3
+
+   application_worker:
+       DEFAULT_EXCHANGE_RATE:
+           KRW: 1178.7
+           JPY: 114.2
+           CNY: 6.3
+
+    volumeMounts:
+        application: []
+        application_worker: []
+        application_scheduler: []
+        application_rest: []
+
+    pod:
+        spec: {}
```
### database.yaml
- [ADD] New database - cost-analysis
```diff
+cost-analysis:
+    database:
+        DATABASES:
+            default:
+              username: ___CHANGE_YOUR_DB_USERNAME___
+              password: ___CHANGE_YOUR_DB_PASSWORD___
+              db: cost-analysis
+              host: ___CHANGE_YOUR_DOCDB_CLUSTER_HOSTNAME___
+              port: 27017
+              ssl: False
+              read_preference: PRIMARY
+              maxPoolSize: 200
+        CACHES:
+            default:
+              backend: spaceone.core.cache.redis_cache.RedisCache
+              host: redis
+              port: 6379
+              db: 15
+              encoding: utf-8
+              socket_timeout: 10
+              socket_connect_timeout: 10

```
## DB
### index reset
- apply all databases
```
use {database_name};
```
```
db.getCollectionNames().forEach(function(collName) { 
    db.runCommand({dropIndexes: collName, index: "*"});
});
```

### update schema - inventory
```
use inventory
```
```
db.server.updateMany({}, {$unset: {"collection_info.jobs": ""}})
db.server.updateMany({}, {$unset: {"collection_info.state": ""}})
db.server.updateMany({}, {$unset: {"garbage_collection": ""}})
db.server.updateMany({}, {$unset: {"server_type": ""}})

db.cloud_service.updateMany({}, {$unset: {"collection_info.jobs": ""}})
db.cloud_service.updateMany({}, {$unset: {"collection_info.state": ""}})
db.cloud_service.updateMany({}, {$unset: {"garbage_collection": ""}})

db.cloud_service_type.updateMany({}, {$unset: {"collection_info": ""}})
db.cloud_service_type.updateMany({}, {$unset: {"garbage_collection": ""}})

db.server.updateMany({state: "INSERVICE"}, {$set: {state: "ACTIVE"}})
db.cloud_service.updateMany({state: "INSERVICE"}, {$set: {state: "ACTIVE"}})
```
### update schema - config
```
use config
```
```
db.user_config.deleteMany({name: /table$/})
```


## Upgrade helm chart

~~~
kcd spaceone
helm repo update

# If you use Type 1.
helm upgrade spaceone -f values.yaml -f frontend.yaml spaceone/spaceone

# if you use Type 2.
helm upgrade spaceone -f values.yaml -f frontend.yaml -f database.yaml spaceone/spaceone
~~~
