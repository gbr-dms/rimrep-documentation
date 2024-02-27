# Infrastructure as Code

WIP

## Summary

- External services + Cloud infrastructure + Kubernetes bootstrap: [`rimrep-terraform`](https://github.com/aodn/rimrep-terraform)
- GitOps: [`rimrep-flux`](https://github.com/aodn/rimrep-flux)

## Cluster overview

### Internal Services

See [Internal tools and services](./../../internal/internal-tools-and-services.md)

```mermaid
%%{
  init: {
    'theme': 'base',
    'themeVariables': {
      'edgeLabelBackground': '#ffffff',
      'tertiaryTextColor': '#0f00aa',
      'clusterBkg': '#fafaff',
      'clusterBorder': '#0f00aa'
    }
  }
}%%

flowchart LR
  classDef red fill:#ffcccc,stroke:#ff0000;

  subgraph terraform["Terraform"]
    direction LR
    infrastructure(Infrastructure Workspace)
    network(Network Workspace)
  end

  subgraph "Okta"
    aodn_tenant("AODN tenant")
  end

  subgraph "Slack"
    slack("RIMReP DMS Workspace")
  end

  subgraph github["GitHub"]
    data_pipeline_repo(rimrep-data-pipeline)
    stac_fastapi_repo(rimrep-stac-fastapi)
    flux_repo(rimrep-flux)
  end

  subgraph aws["AWS Managed Services"]
    route53("AWS Route53")
    lb("AWS NLB ")

    rds("AWS RDS Postgres")
    s3("AWS S3")
    ecr("AWS ECR")

    subgraph eks["EKS"]
      subgraph "Fargate Nodes"
        flux(Flux)
        karpenter(Karpenter)
        coredns(CoreDNS)
        aws_lb_controller(AWS Load Balancer Controller)
      end

      subgraph "EC2 Nodes"
        cert_manager(cert-manager)
        aws_external_dns(aws-external-dns)
        ingress_nginx(ingress-nginx Controller)

        subgraph prometheus_stack["Prometheus Stack"]
          alertmanager(Alertmanager)
          prometheus(Prometheus)
          loki(Loki)
          grafana(Grafana)
        end

        argo_workflows(Argo Workflows)
        ww_gitops(Weave GitOps)
        internal_stac_fastapi(internal-stac-fastapi)
      end
    end
  end

  terraform --> aws & flux_repo

  stac_fastapi_repo & data_pipeline_repo --> ecr

  flux_repo <--> flux

  ecr --> argo_workflows & internal_stac_fastapi

  users((Internal users))

  route53 <--> aws_external_dns

  rds <--> grafana & argo_workflows & internal_stac_fastapi

  lb --> users

  s3 <--> argo_workflows

  argo_workflows & grafana --> lb

  users <--> aodn_tenant

  alertmanager ---> slack

```

## External Services

```mermaid
%%{
  init: {
    'theme': 'base',
    'themeVariables': {
      'edgeLabelBackground': '#ffffff',
      'tertiaryTextColor': '#0f00aa',
      'clusterBkg': '#fafaff',
      'clusterBorder': '#0f00aa'
    }
  }
}%%

flowchart LR
  classDef red fill:#ffcccc,stroke:#ff0000;

  subgraph terraform["Terraform"]
    direction LR
    infrastructure(Infrastructure Workspace)
  end

  subgraph github["GitHub"]
    direction LR
    pygeoapi_repo(rimrep-pygeoapi)
    stac_api_repo(rimrep-stac-fastapi)
    stac_browser_repo(rimrep-stac-browser)
    dashboard_repo(rimrep-dashboard)
    metcalf_repo(rimrep-metcalf)
  end

  subgraph auth0["Auth0"]
    tenant("Tenant")
  end

  subgraph aws["AWS Managed Services"]
    lb("AWS NLB ")
    rds("AWS RDS Postgres")
    s3("AWS S3")
    ecr("AWS ECR")

    subgraph eks["EKS"]
      subgraph "EC2 Nodes"
        data_api(pygeoapi)
        s3_proxy(S3 Proxy):::red
        oauth2_proxy(Oauth2 Proxy):::red

        stac_fastapi(stac-fastapi)

        metadata_frontend(stac-browser)
        dashboard(dashboard)

        metcalf(metcalf)
      end
    end
  end

  terraform ------> aws & tenant

  pygeoapi_repo & stac_api_repo & stac_browser_repo & dashboard_repo & metcalf_repo --> ecr
  ecr ---> data_api & stac_fastapi & metadata_frontend & dashboard & metcalf

  users((External Users))

  data_api & stac_fastapi & metadata_frontend & dashboard--> oauth2_proxy

  rds ---> stac_fastapi
  rds <--> metcalf

  s3 --> s3_proxy & data_api
  s3 --> users
  s3_proxy --> data_api & oauth2_proxy

  oauth2_proxy --> lb --> users

  metcalf --> lb

  users <--> tenant

```
