# Kubernetest namespace

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

read pre-condition/README.md

update values in pre-conditon
apply pre-condition

# Install

* update values.yaml
* update database.yaml 
* update frontend.yaml


If you use mongodb cluster,
host is "localhost" in database.yaml
Use TYPE 2. global varable in values.yaml

~~~
kcd spaceone

helm install spaceone -f values.yaml -f database.yaml -f frontend.yaml spaceone/spaceone

~~~


# Upgrade

From v1.6.7 to v1.7.1, there is three key changes in values.yaml

## application

the ***application*** keyword is changed to application_grpc.
In a values.yaml, change every application to application_grpc

* AS-IS

~~~
secret:
    enabled: true
    replicas: 1
    image:
      name: public.ecr.aws/megazone/spaceone/secret
      version: 1.7.1
    application:
        CONNECTORS:
            AWSSecretManagerConnector:
                aws_access_key_id: ___CHANGE_YOUR_ACCESS_KEY_ID___
                aws_secret_access_key: __CHANGE_YOUR_SECRET_ACCESS_KEY__
                region_name: __AWS_REGION_NAME__
~~~

* NOW

~~~
secret:
    enabled: true
    replicas: 1
    image:
      name: public.ecr.aws/megazone/spaceone/secret
      version: 1.7.1
    application_grpc:
        BACKEND: AWSSecretManagerConnector
        CONNECTORS:
            AWSSecretManagerConnector:
                aws_access_key_id: ___CHANGE_YOUR_ACCESS_KEY_ID___
                aws_secret_access_key: __CHANGE_YOUR_SECRET_ACCESS_KEY__
                region_name: __AWS_REGION_NAME__
~~~

## BACKEND in secret

In a secret configuration, add BACKEND which is the indicator of CONNECTORS.
If you use AWSSecretManagerConnector, add BACKEND as ***AWSSecretManagerConnector***

* NOW

~~~
secret:
    enabled: true
    replicas: 1
    image:
      name: public.ecr.aws/megazone/spaceone/secret
      version: 1.7.1
    application_grpc:
        BACKEND: AWSSecretManagerConnector
        CONNECTORS:
            AWSSecretManagerConnector:
                aws_access_key_id: ___CHANGE_YOUR_ACCESS_KEY_ID___
                aws_secret_access_key: __CHANGE_YOUR_SECRET_ACCESS_KEY__
                region_name: __AWS_REGION_NAME__
~~~

## service (Backend services only, not console, console-api)

The service values in every charts has two more sections ***grpc*** and ***rest***

Add one more step, ***grpc***

* ASIS

~~~
    service:
      type: NodePort
      ports:
        - name: grpc
          port: 50051
          targetPort: 50051
          nodePort: 30102
          protocol: TCP
~~~

* NOW

~~~
    service:
      grpc:
        type: NodePort
        ports:
          - name: grpc
            port: 50051
            targetPort: 50051
            nodePort: 30102
            protocol: TCP
~~~


* Upgrade helm chart


~~~
kcd spaceone
helm upgrade spaceone -f values.yaml -f database.yaml -f frontend.yaml spaceone/spaceone

~~~
