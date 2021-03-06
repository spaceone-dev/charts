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

# Pre-condition

~~~
git clone https://github.com/spaceone-dev/charts.git
cd charts
cd example/v1.7.2/pre-condition
kubectl create -f shared.yaml
~~~

read pre-condition/README.md

update values in pre-conditon
apply pre-condition

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

There is one configmap change between v1.7.1 and v1.7.2.

File: pre-condition/shared.yaml

* Add AuthenticationAPIKeyHandler in authentication

~~~
         authentication:
         - backend: spaceone.core.handler.authentication_handler.AuthenticationGRPCHandler
           uri: grpc://identity:50051/v1/Domain/get_public_key
+        - backend: spaceone.core.handler.authentication_api_key_handler.AuthenticationAPIKeyHandler
+          uri: grpc://identity:50051/v1/APIKey/get
         authorization:
         - backend: spaceone.core.handler.authorization_handler.AuthorizationGRPCHandler
           uri: grpc://identity:50051/v1/Authorization/verify
~~~

After update shared.yaml

~~~sh
kubectl apply -f shared.yaml
~~~

There is no change in values files. ***%s/1.7.1/1.7.2/g*** in VIM.
Just update your docker image version number of each service.

* Upgrade helm chart

~~~
kcd spaceone
helm repo update

# If you use Type 1.
helm upgrade spaceone -f values.yaml -f frontend.yaml spaceone/spaceone

# if you use Type 2.
helm upgrade spaceone -f values.yaml -f frontend.yaml -f database.yaml spaceone/spaceone
~~~
