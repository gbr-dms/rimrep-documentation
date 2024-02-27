
# New Relic Onboarding README

Welcome to the New Relic Onboarding guide! This document aims to assist you in getting started with New Relic by providing step-by-step instructions for account setup, Kubernetes cluster integration, dashboard creation and management, alerts configuration, and logs management.

## Table of Contents
1. [Account Onboarding](#account-onboarding)
2. [Kubernetes Clusters Onboarding](#kubernetes-clusters-onboarding)
    - [Namespace Onboarding for Metrics](#namespace-onboarding-for-metrics)
    - [Logs Enablement by Namespace](#logs-enablement-by-namespace)
3. [Dashboards Management](#dashboards-management)
4. [Alerts Management](#alerts-management)
5. [Logs Management](#logs-management)

---

## Account Onboarding

To begin using New Relic, follow these steps to set up your account through IaC:

1. Add newrelic terraform provider, account_id, api_key and region
```hcl
terraform {
  required_providers {
    newrelic = {
      source = "newrelic/newrelic"
      version = "3.28.1"
    }
  }
}
provider "newrelic" {
  account_id = split(":", data.aws_secretsmanager_secret_version.newrelic_account_creds.secret_string)[0]
  api_key = split(":", data.aws_secretsmanager_secret_version.newrelic_account_creds.secret_string)[1]    # Usually prefixed with 'NRAK'
  region = var.newrelic_account_region                    # Valid regions are US and EU
}

Note: Make sure the account credentials are available in AWS Secret manager in the respective environments before running the stack.

```

---

## Kubernetes Clusters Onboarding

1. Navigate to the module `modules\monitoring\main.tf`.
2. Locate the resource block for `helm_release` named `"newrelic_bundle"`.
3. Set the flag `enable_newrelic_bundle` to `**true**` to enable Kubernetes clusters onboarding.

### Namespace Onboarding for Metrics

To onboard a namespace for metrics monitoring, follow these steps:
1. Add the label `newrelic.com/scrape` with the value `"**true**"` to the namespace.
   Note: Remove the label or set it to **false**, to remove it from the monitoring.

### Logs Enablement by Namespace

To enable logs for a particular namespace, follow these steps:

1. Set the variable `log_namespace_list` with the list of namespaces where logging should be enabled. For example:
   ```hcl
   log_namespace_list = "/var/log/containers/*ingress-nginx*.log,/var/log/containers/*pygeoapi*.log,/var/log/containers/*stac-fastapi*.log,/var/log/containers/*metcalf*.log"
   ```

2. If you need to disable logging entirely, set the variable `logging_enabled` to `"false"`.

---

## Dashboards Management

Dashboards are managed through the New Relic UI as it's more convenient due to constant changes and JSON upgrades. Below is the list of Dashboards Enabled:

- Kubernetes Dashboard: Provides an overview of your Kubernetes platform health. Apply filters to focus on a specific cluster or namespace. Container restarts are usually not a good thing, this dashboard will help identify the biggest offenders and where they are currently trying to run. Over-Utilization Indicators In some environments we may be over-utilizing key resources. In those scenarios optimization is contra-indicated. CPU Thottling and OOM Kills are good indicators of potential overutilization of CPU and Memory.

- RDS Dashboard: In this dashboard you can add various widgets to visualize different metrics. Some common RDS metrics you might want to include are CPU utilization, memory utilization, disk I/O, query performance, network throughput, latency, free storage space, database connections, and so on.

- Nginx Ingress Dashboard: The NGINX ingress controller exposes many useful metrics that can be scraped by a Prometheus server or agent. These metrics can inform you of ongoing process connections and the current ingress load that your system is handling. Furthermore, these metrics can inform you on ingress config reloads and empower alert conditions on config reload errors or unexpected config changes.

- Prometheus Dashboard: This dashboard contains charts that provide information about metrics volume sent to New Relic. These charts are useful in helping to monitor the metrics you are sending to the New Relic Remote Write.

Note: Custom dashboards are managed through Terraform.

---

## Alerts Management

Configuring alerts in New Relic allows you to receive notifications when certain conditions are met. Follow these steps to set up and manage alerts in IaC:

1. Module for alerts notification
2. NGINX `nginx_alert_conditions`
3. RDS `rds_alert_conditions`
4. EKS `eks_alert_conditions`
```hcl
module "rimrep_newrelic_monitoring" {
  source                  = "../../modules/newrelic"
  newrelic_account_id     = split(":", data.aws_secretsmanager_secret_version.newrelic_account_creds.secret_string)[0]
  newrelic_account_region = var.newrelic_account_region
  name                    = var.name
  nginx_alert_conditions  = var.nginx_alert_conditions
  rds_alert_conditions    = var.rds_alert_conditions
  eks_alert_conditions    = var.eks_alert_conditions
}
```
5. Add alert conditions dynamically for each infrastructure resources (RDS, EKS, NGINX etc...)in `environment/monitoring/newrelic.auto.tfvars`
6. Add NRQL `query` to set alerts for specific namespaceName/clusterName
7. Set threshold for critical/warning
```hcl
{
    name                            = "eks-pods-failing"
    description                     = "Alert when more than 5 pods are failing in a namespace for more than 5 minutes"
    query                           = "from K8sPodSample select uniqueCount(podName) where status = 'Failed' facet namespaceName, clusterName"
    critical_operator               = "above"
    critical_threshold              = 0
    critical_threshold_duration     = 300
    warning_operator                = "above"
    warning_threshold               = 1
    warning_threshold_duration      = 300
  }
```

---

## Muting Rules
Use muting rules to mute notifications when you don't need them, like when you're performing maintenance. 

1. Set mute rules for alerts
2. Mute rules are configured under `module/newrelic/main.tf`
3. Mute rules are set for for EKS,RDS & NGINX, which can be disabled/enabled
```hcl
resource "newrelic_alert_muting_rule" "alert_mute" {
    name = "newrelic-muting-rule-${var.name}"
    enabled = var.mute_rule_values.enabled
    description = "mute alerts for specified schedule"
    condition {
        conditions {
            attribute   = "policyId"
            operator    = "IN"
            values      = [newrelic_alert_policy.rds_alert_policy.id, newrelic_alert_policy.eks_alert_policy.id, newrelic_alert_policy.nginx_ingress_controller_policy.id]
        }
        operator = "AND"
    }
    schedule {
      start_time = var.mute_rule_values.start_time
      end_time = var.mute_rule_values.end_time
      time_zone = var.mute_rule_values.time_zone
      repeat = var.mute_rule_values.repeat
      repeat_count = var.mute_rule_values.repeat_count
    }
}
```
4. Mute rules can be enabled/scheduled for alert notifications.
```hcl
{
    enabled             = true
    start_time          = "2024-02-02T16:00:00"
    end_time            = "2024-02-05T13:30:00"
    time_zone           = "Australia/Sydney"
    repeat              = "WEEKLY"
    repeat_count        = 42
}
```

## Additional Resources

For further assistance or detailed documentation, refer to the following resources:

- https://docs.newrelic.com/docs/kubernetes-pixie/kubernetes-integration/get-started/introduction-kubernetes-integration/

---



