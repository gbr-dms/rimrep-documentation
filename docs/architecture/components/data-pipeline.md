# Data pipeline

See [Data pipeline requirements](../../requirements.md#data-pipeline)

## Summary

- Argo Workflow
  - Workflows and tempaltes are stored and synced in the [`rimrep-flux`](https://github.com/aodn/rimrep-flux/tree/main/workflows) GitHub repository.
  - Internal auth handled with Okta (maintained by AODN)
- Python modules and scripts: [`rimrep-data-pipeline`](https://github.com/aodn/rimrep-data-pipeline)
- For data storage and auth see [Data system architecture](data-system.md)

## Architecture

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

  subgraph "AWS S3"
    public_data_storage[(Public data)]
    artifact_storage[(Artifacts/Logs)]:::red
    private_data_storage[(Private data)]:::red
  end

  aws_ecr[(AWS ECR)]
  aws_rds[(STAC DB)]

  subgraph argo_workflows["Argo Workflows"]
    data_workflow(Data workflows)

    metadata_workflow(Metadata workflows)
  end

  subgraph group_k8s ["k8s"]
    stac_fastapi(stac-fastapi)
    internal_stac_fastapi(internal-stac-fastapi)
    pygeoapi(pygeoapi)
  end

  rimrep_admin((RIMReP Admin))

  data_pipeline_repo -->|Push container images to| aws_ecr
  aws_ecr -->|Pull container images from| argo_workflows
  argo_repo -->|Sync workflows/templates| argo_workflows

  data_workflow -->|ARCO data| public_data_storage & private_data_storage
  metadata_workflow & data_workflow -->|Artifacts/logs| artifact_storage

  catalog -. Jsonnet files .-> metadata_workflow

  data --->|Ingest| data_workflow
  metadata -. Harvest .-> metadata_workflow
  rimrep_admin -. "Curate initial datapackage.json<br>& Jsonnet files" .-> catalog
  catalog -. "Initial datapackage.json" .-> data_workflow
  data_workflow -. "populated datapackage.json with<br>data-driven metadata" .-> metadata_workflow
  metadata_workflow -. "complete datapackage.json" .-> public_data_storage
  metadata_workflow -. "complete datapackage.json" .-> private_data_storage
  metadata_workflow -. Generate STAC Collection&Item .-> internal_stac_fastapi
  internal_stac_fastapi -. Writes STAC to .-> aws_rds
  aws_rds -. Reads STAC from .-> stac_fastapi
  data_workflow -. Generate entries for .-> pygeoapi
```

### Argo Workflows

Kubernetes native workflow engine. Basically DAG of container jobs/steps.

Workflows and workflow templates are stored and synced in the [`rimrep-flux`](https://github.com/aodn/rimrep-flux/tree/main/workflows) GitHub repository.

All temporary artifacts and logs are stored in an AWS S3 `rimrep-argowf-artifacts` bucket.

### Data pipeline repository

The [`rimrep-data-pipeline`](https://github.com/aodn/rimrep-data-pipeline) contains multiple python modules and scripts that are used to process data and metadata. It publishes several docker container images to AWS ECR.

## Auth

Argo Workflows is for internal use only. Authentication is handled through AODN's Okta instance. See [Internal tools and services](../../internal/internal-tools-and-services.md#argo-workflows)
