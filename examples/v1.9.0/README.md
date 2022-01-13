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
- 1.9.0
- \+ (hotfix) cost-analysis 1.9.0.1

### value.yaml
- (hotfix)[ADD] console.ingress.annotations
```diff
  ingress:
    enabled: true
    host: '*.console.spaceone.dev'   # host for ingress (ex. *.console.spaceone.dev)
    annotations:
      kubernetes.io/ingress.class: alb
      alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS":443}]'
      alb.ingress.kubernetes.io/actions.ssl-redirect: '{"Type": "redirect", "RedirectConfig": { "Protocol": "HTTPS", "Port": "443", "StatusCode": "HTTP_301"}}'
      alb.ingress.kubernetes.io/inbound-cidrs: 0.0.0.0/0 # replace or leave out
      alb.ingress.kubernetes.io/scheme: "internet-facing" # internet-facing
      alb.ingress.kubernetes.io/target-type: instance # Your console and console-api should be NodePort for this configuration.
      alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-northeast-2:111111111111:certificate/11111111-1111-111111111-111111111111
+     alb.ingress.kubernetes.io/healthcheck-path: "/check"
+     alb.ingress.kubernetes.io/load-balancer-attributes: idle_timeout.timeout_seconds=60 # default 60
      external-dns.alpha.kubernetes.io/hostname: "*.console.spaceone.dev"
```

- (hotfix)[ADD] console-api.ingress.annotations
```diff
  ingress:
    enabled: true
    host: 'console-api.spaceone.dev'   # host for ingress (ex. console-api.spaceone.dev)
    annotations:
        kubernetes.io/ingress.class: alb
        alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS":443}]'
        alb.ingress.kubernetes.io/actions.ssl-redirect: '{"Type": "redirect", "RedirectConfig": { "Protocol": "HTTPS", "Port": "443", "StatusCode": "HTTP_301"}}'
        alb.ingress.kubernetes.io/inbound-cidrs: 0.0.0.0/0 # replace or leave out
        alb.ingress.kubernetes.io/scheme: "internet-facing" # internet-facing
        alb.ingress.kubernetes.io/target-type: instance # Your console and console-api should be NodePort for this configuration.
        alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-northeast-2:111111111111:certificate/11111111-1111-1111-1111-111111111111
+       alb.ingress.kubernetes.io/healthcheck-path: "/check"
+       alb.ingress.kubernetes.io/load-balancer-attributes: idle_timeout.timeout_seconds=60 # default 60
        external-dns.alpha.kubernetes.io/hostname: console-api.spaceone.dev
```

- [ADD] cost-analysis.application_scheduler.TOKEN

```diff
cost-analysis:
    enabled: true
    scheduler: true
    worker: true
    replicas: 1
    replicas_worker: 2
    image:
      name: public.ecr.aws/megazone/spaceone/cost-analysis
      version: 1.9.0.1

+   # Overwrite scheduler config
+   application_scheduler:
+       TOKEN: <root_token>

    application_grpc:
        DEFAULT_EXCHANGE_RATE:
            KRW: 1178.7
            JPY: 114.2
            CNY: 6.3
```

### root-domain.yaml(Only when creating the initial root domain)
```diff
enabled: true
image:
    name: spaceone/spacectl
    version: 1.9.0
domain: root
main:
  import:
-   - /root/spacectl/apply/root_domain.yaml
-   - /root/spacectl/apply/domain_role.yaml
-   - /root/spacectl/apply/project_role.yaml
-   - /root/spacectl/apply/repository.yaml
-   - /root/spacectl/apply/schema.yaml
-   - /root/spacectl/apply/register_plugins.yaml
+   - /root/spacectl/apply/root_domain.yaml 
+   - /root/spacectl/apply/marketplace.yaml
+   - /root/spacectl/apply/hyperbilling.yaml # If you have a hyperbilling account
  var:
    domain_name: root
    domain_owner:
      id: admin
      password: Adminpassword
    user:
      id: root_api_key
-   username: user1@example.com
-   password: User1234!
    consul_server: spaceone-consul-server
-   plugin_repo : spaceone
+   marketplace_endpoint: grpc://repository.portal.spaceone.dev:50051
    tasks: []
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
