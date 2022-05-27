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
- 1.9.7
    - identity : 1.9.7.2
    - plugin: 1.9.7.1
    - console-api: 1.9.7.1

### values.yaml
- [ADD] notification.application_grpc
```diff
# ex: 'PROTOCOL_PLUGIN_ID': {'month': QUOTA, 'day': QUOTA},
+        DEFAULT_QUOTA:
+            plugin-sms-notification-protocol:
+              month: 10000
+            plugin-voicecall-notification-protocol:
+              month: 1000

```
- [DELETE] billing(deprecated)
```diff
-billing:
-    enabled: false
-    replicas: 1
-    image:
-      name: public.ecr.aws/megazone/spaceone/billing
-      version: 1.9.7
-
-    pod:
-        spec: {}
```

### Delete hyperbilling service_account, provider
- delete secret data
```
spacectl list secret.Secret -p provider=hyperbilling -p domain_id=<domain_id>
spacectl exec delete secret.Secret -p secret_id=<secret_id> -p domain_id=<domain_id>
```
- delete service account
```
spacectl list service_account -p provider=hyperbilling -p domain_id=<domain_id>
spacectl exec delete service_account -p service_account_id=<service_account_id> -p domain_id=<domain_id>
```
- delete provider
```
spacectl exec delete provider -p provider=hyperbilling -p domain_id=<domain_id>
```

### Update policy
> Before proceeding with this you must ensure that the role using 'policy-0386cce2730b' exists and delete it.

- delete Managed policy
```
spacectl exec delete repository.Policy -p policy_id=policy-managed-alert-manager-full-access

spacectl exec delete repository.Policy -p policy_id=policy-0386cce2730b
```

- create new managed policy
```
TODO
```

- Update roles for all domains
```
script
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
