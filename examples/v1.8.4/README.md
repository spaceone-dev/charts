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
### value.yaml
- [ADD] inventory.application_worker.HANDLERS
```diff
    application_worker:
#        TOKEN: ___CHANGE_YOUR_ROOT_TOKEN___ 
        TOKEN_INFO:
            protocol: consul
            config:
                host: spaceone-consul-server
            uri: root/api_key/TOKEN
+       HANDLERS:
+         authentication: []
+         authorization: []
+         mutation: []
```
- [DELETE] repository.application_grpc
```diff
repository:
    enabled: true
    replicas: 1
    image:
      name: public.ecr.aws/megazone/spaceone/repository
      version: 1.8.4
-    application_grpc:
-#        ROOT_TOKEN: ___CHANGE_YOUR_ROOT_TOKEN___ 
-        ROOT_TOKEN_INFO:
-            protocol: consul
-            config:
-                host: spaceone-consul-server
-            uri: root/api_key/TOKEN
```
### frontend.yaml
- [ADD] console-api.production_json.escalation
```diff
  production_json:
      cors:
      - http://*
      - https://*
#      - https://*.console.spaceone.dev
#      - https://*.console.spaceone.dev:8080
      redis:
          host: redis
          port: 6379
          db: 15
      logger:
          handlers:
          - type: console
            level: debug
          - type: file
            level: info
            format: json
            path: "/var/log/spaceone/console-api.log"
+     escalation:
+       enabled: false
+       allowedDomainId: #root-domain
+       apiKey: #root-api-key
```

### Upgrade helm chart

~~~
kcd spaceone
helm repo update

# If you use Type 1.
helm upgrade spaceone -f values.yaml -f frontend.yaml spaceone/spaceone

# if you use Type 2.
helm upgrade spaceone -f values.yaml -f frontend.yaml -f database.yaml spaceone/spaceone
~~~
