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
- 1.9.1
- \+ (hotfix) identity 1.9.1.1
- \+ (hotfix) console 1.9.1.1

### frontend.yaml
- [ADD] console.production_json.CONTACT_LINK
```diff
console:
  enabled: true
  developer: false
  name: console
  replicas: 1
  image:
      name: public.ecr.aws/megazone/spaceone/console
      version: 1.9.1.1
  imagePullPolicy: IfNotPresent

...
  production_json:
    CONSOLE_API:
        ENDPOINT: https://console-api.spaceone.dev
+   CONTACT_LINK: <enter_your_site_url_or_email> # Change the contact us link of sign in page
```
- [ADD] console-api.production_json.billing
```diff
console-api:
  enabled: true
  developer: false
  name: console-api
  replicas: 1
  image:
      name: public.ecr.aws/megazone/spaceone/console-api
      version: 1.9.1
  imagePullPolicy: IfNotPresent
...
  production_json:
...
      escalation:
          enabled: false
          allowedDomainId: #root-domain
          apiKey: #root-api-key
+     billing:  #Change billing data source in main and project dashboards
+         source: v2
        
```

### value.yaml
- [add] monitoring.ingress.rest
```diff
monitoring:
    enabled: true
    grpc: true
    scheduler: true
    worker: true
    rest: true
...
    ingress:
+     rest:
+       enabled: true
        annotations:
            kubernetes.io/ingress.class: alb
            alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS":443}]'
            alb.ingress.kubernetes.io/inbound-cidrs: 0.0.0.0/0 # replace or leave out
            alb.ingress.kubernetes.io/scheme: internet-facing
            alb.ingress.kubernetes.io/target-type: ip 
            alb.ingress.kubernetes.io/certificate-arn: "arn:aws:acm:ap-northeast-2:111111111111:certificate/11111111-458f-4c5f-b4c5-e3a2eb34caff"
            alb.ingress.kubernetes.io/healthcheck-path: "/check"
            alb.ingress.kubernetes.io/load-balancer-attributes: idle_timeout.timeout_seconds=600
            external-dns.alpha.kubernetes.io/hostname: monitoring-webhook.example.com
        servicePort: 80
        path: /*
```
### DB patch
- cost-analysis
```
use cost-analysis;

db.cost_query_set.updateMany({}, {$unset: {scope: ""}})
db.cost_query_set.dropIndex('scope_1')

db.dashboard.renameCollection('public_dashboard')
db.public_dashboard.dropIndex('scope_1')
db.public_dashboard.dropIndex('user_id_1')
db.public_dashboard.dropIndex('dashboard_id_1')
db.public_dashboard.updateMany({}, {$unset: {scope: ""}})
db.public_dashboard.updateMany({}, {$unset: {user_id: ""}})
db.public_dashboard.updateMany({}, {$rename: {dashboard_id: "public_dashboard_id"}})
```

### root-domain.yaml(Only when creating the initial root domain)
- [ADD] default_language 
- [ADD] default_timezone
- [HOTFIX] Add
```diff
enabled: true
image:
    name: spaceone/spacectl
    version: 1.9.1
domain: root
main:
  import:
    - /root/spacectl/apply/root_domain.yaml 
    - /root/spacectl/apply/marketplace.yaml
    - /root/spacectl/apply/hyperbilling.yaml # If you have a hyperbilling account
  var:
    domain_name: root
+   default_language: ko
+   default_timezone: Asia/Seoul
    domain_owner:
      id: admin
      password: Adminpassword
    user:
      id: root_api_key
    consul_server: spaceone-consul-server
    marketplace_endpoint: grpc://repository.portal.spaceone.dev:50051
+   project_admin_policy_type: MANAGED
+   project_admin_policy_id: policy-managed-project-admin
+   project_viewer_policy_type: MANAGED
+   project_viewer_policy_id: policy-managed-project-viewer
+   domain_admin_policy_type: MANAGED
+   domain_admin_policy_id: policy-managed-domain-admin
+   domain_viewer_policy_type: MANAGED
+   domain_viewer_policy_id: policy-managed-domain-viewer
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
