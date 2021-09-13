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
- [ADD] monitoring.application_grpc.INSTALLED_DATA_SOURCE_PLUGINS
```diff
  monitoring:
  ...
      application_grpc:
          WEBHOOK_DOMAIN: https://monitoring-webhook.spaceone.dev
          TOKEN: []
+          INSTALLED_DATA_SOURCE_PLUGINS:
+            - name: AWS CloudWatch
+              plugin_info:
+                plugin_id: plugin-41782f6158bb
+                provider: aws
+            - name: Azure Monitor
+              plugin_info:
+                plugin_id: plugin-c6c14566298c
+                provider: azure
+            - name: Google Cloud Monitoring
+              plugin_info:
+                plugin_id: plugin-57773973639a
+                provider: google_cloud

```
- [ADD] notification.application_grpc.INSTALLED_DATA_SOURCE_PLUGINS
```diff
monitoring:
...
    application_grpc:
        WEBHOOK_DOMAIN: https://monitoring-webhook.spaceone.dev
        TOKEN: []
+        INSTALLED_PROTOCOL_PLUGINS:
+          - name: Slack
+            plugin_info:
+              plugin_id: slack-notification-protocol
+              options: {}
+              schema: slack_webhook
+          - name: Telegram
+            plugin_info:
+              plugin_id: plugin-telegram-noti-protocol
+              options: {}
+              schema: telegram_auth_token
+          - name: Email
+            plugin_info:
+              plugin_id: plugin-email-noti-protocol
+              options: {}
+              secret_data:
+                smtp_host: email-smtp.us-west-2.amazonaws.com
+                smtp_port: "587"
+                user: **********
+                password: **********
+              schema: email_smtp
```
- Upgrade helm chart

~~~
kcd spaceone
helm repo update

# If you use Type 1.
helm upgrade spaceone -f values.yaml -f frontend.yaml spaceone/spaceone

# if you use Type 2.
helm upgrade spaceone -f values.yaml -f frontend.yaml -f database.yaml spaceone/spaceone
~~~
