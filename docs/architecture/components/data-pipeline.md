# Data pipeline

See [Data pipeline requirements](../../requirements.md#data-pipeline)

## Summary

- Argo Workflow
  - Workflows and templates are stored in the [`rimrep-argo-workflow`](https://github.com/aodn/rimrep-argo-workflow/tree/main/workflows) GitHub repository.
  - Workflows are synced in [`rimrep-flux`](https://github.com/aodn/rimrep-flux) GitHub repository.
  - Internal auth handled with Okta (maintained by AODN)
- Python modules and scripts: [`rimrep-data-pipeline`](https://github.com/aodn/rimrep-data-pipeline)
- For data storage and auth see [Data system architecture](data-system.md)

## Architecture

### High level architecture of the pipeline

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

flowchart TB
  classDef red fill:#ffcccc,stroke:#ff0000;

  subgraph external["External"]
    data[(Data)]
    metadata[(Metadata)]
  end

  subgraph "GitHub"
    catalog(rimrep-catalog)
    argo_repo(rimrep-argo-workflow)
    data_pipeline_repo(rimrep-data-pipeline)
  end

  subgraph group_s3["AWS S3"]
    direction LR
    public_data_storage[(Public data)]
    private_data_storage[(Private data)]:::red
    artifact_storage[(Artifacts/Logs)]:::red
  end

  aws_ecr[(AWS ECR)]

  subgraph group_k8s ["k8s"]
    stac_fastapi(stac-fastapi)
    pygeoapi(pygeoapi)
    
    subgraph argo_workflows["Argo Workflows"]
      data_workflow(Dataset workflows)
    end
  end

  group_k8s ~~~ group_s3

  rimrep_admin((RIMReP Admin))

  data_pipeline_repo -->|Push container images to| aws_ecr
  aws_ecr -->|Pull container images from| argo_workflows
  argo_repo -->|Sync workflows/templates| argo_workflows

  data_workflow -->|ARCO data\nFrictionless datapackage.json\n& tableschema.json/gridscheme.json| public_data_storage & private_data_storage
  data_workflow --> |Artifacts/logs|artifact_storage

  catalog --> |tableschema.json/gridschema.json\ncollection.jsonnet\n*.libsonnet| data_workflow
  catalog <--> |datapackage.json| data_workflow

  data --->|Ingest| data_workflow
  metadata --> |Harvest|data_workflow
  rimrep_admin --> |Curate dataset issues\n& collection.jsonnet|catalog
  data_workflow --> |Updates| pygeoapi
  data_workflow --> |Updates| stac_fastapi

```
### Argo Workflows

Kubernetes native workflow engine. Basically DAG of container jobs/steps.

Workflows and templates are stored in the [`rimrep-argo-workflow`](https://github.com/aodn/rimrep-argo-workflow/tree/main/workflows) GitHub repository.

Workflows are synced in [`rimrep-flux`](https://github.com/aodn/rimrep-flux) GitHub repository.

All temporary artifacts and logs are stored in an AWS S3 `rimrep-argowf-artifacts` bucket.

### Data pipeline repository

The [`rimrep-data-pipeline`](https://github.com/aodn/rimrep-data-pipeline) contains multiple Python modules and scripts that are used to process data and metadata. It publishes several docker container images to AWS ECR.

## Auth

Argo Workflows is for internal use only. Authentication is handled through AODN's Okta instance.
