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
    data_workflow(Dataset argo workflows)
  end

  group_k8s ~~~ group_s3

  rimrep_admin((RIMReP Admin))

  data_pipeline_repo -->|Push container images to| aws_ecr
  aws_ecr -->|Pull container images from| data_workflow
  argo_repo -->|Sync workflows/templates| data_workflow

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

### Flow chart of pipeline steps
```mermaid


flowchart TB
classDef red fill:#ffcccc,stroke:#ff0000;

    source_data[(Source data)]:::red
    coordinates{{Coordinates}}:::red
    
    convert["Convert to parquet/zarr\n(dataset-specific)"]
    check_extents[check-extents]
    has_extents{Can we get spatial/temporal\n extents from the data?}
    harvest_with["harvest-metadata-by-dataset-id\n(with extents)"]
    harvest_without["harvest-metadata-by-dataset-id\n(without extents)"]
    has_updates{has new metadata?}
    update_ds[update-dataset-issue]
    generate_datapkg[generate-datapackage-from-dataset-issue]
    compare_datapkg[compare-datapackage-with-existing]
    datapkg_changed{changed?}
    submit_datapkg[submit-updated-datapkg]
    generate_datadriven[generate-datadriven-metadata]
    file_format{file format?}
    generate_datadriven_parquet[generate-datadriven-metadata-parquet]
    generate_datadriven_zarr[generate-datadriven-metadata-zarr]
    generate_frictionless[generate-frictionless-metadata]
    schema_changed{data schema changed?}
    error((Error.\nData engineer manual check)):::red
    create_pr[create-pr-in-catalog-repo]
    get_catalog[get-catalog-files]
    upload_data[upload-arco-data-to-s3]
    upload_frictionless[upload-frictionless-json-to-s3]
    generate_item[generate-stac-item]
    validate_item[validate-stac-item]
    check_collection[check-stac-collection-exist]
    create_collection[create-stac-collection]
    ingest_initial_collection[ingest-initial-stac-collection]
    collection_exists{collection exists?}
    ingest_item[ingest-stac-item]
    generate_updated_collection[generate-updated-collection-from-items]
    validate_updated_collection[validate-updated-collection]
    ingest_updated_collection[ingest-updated-collection]
    generate_pygeo[generate-pygeoapi-entry]
    validate_pygeo[validate-pygeoapi-entry]
    ingest_pygeo[ingest-pygeoapi-entry]

    s3[(AWS S3)]:::red
    stac[(STAC)]:::red
    pygeoapi[(pygeoapi)]:::red

    source_data --> convert
    convert --> |converted data| generate_datadriven & upload_data
    convert ~~~ check_extents
    coordinates --> generate_datadriven & check_extents
    check_extents --> has_extents
    has_extents --> |yes| harvest_without
    has_extents --> |no| harvest_with
    harvest_without --> has_updates
    harvest_with --> has_updates
    has_updates --> |yes| update_ds
    has_updates --> |no| generate_datapkg
    update_ds --> generate_datapkg
    generate_datapkg --> compare_datapkg
    compare_datapkg --> datapkg_changed
    datapkg_changed --> |yes| submit_datapkg
    submit_datapkg --> create_pr
    datapkg_changed --> |no\n\nbase-branch| get_catalog
    submit_datapkg --> |updated-branch|get_catalog
    get_catalog --> generate_frictionless
    generate_datadriven --> file_format
    file_format --> |parquet|generate_datadriven_parquet
    file_format --> |zarr|generate_datadriven_zarr
    generate_datadriven_parquet & generate_datadriven_zarr --> generate_frictionless
    generate_frictionless --> schema_changed
    schema_changed --> |yes| error
    schema_changed --> |no| upload_data & generate_pygeo & upload_frictionless
    generate_frictionless --> |"datapackage.json\n\n(table/grid)schema.json"|upload_frictionless
    upload_data --> s3
    upload_frictionless --> s3
    generate_pygeo --> validate_pygeo
    validate_pygeo --> ingest_pygeo
    ingest_pygeo --> pygeoapi
    generate_frictionless & get_catalog --> generate_item
    generate_item --> validate_item
    validate_item --> check_collection
    check_collection --> collection_exists
    collection_exists --> |no| create_collection
    create_collection --> ingest_initial_collection
    collection_exists --> |yes| ingest_item
    ingest_initial_collection --> ingest_item & stac
    ingest_item --> generate_updated_collection & stac
    generate_updated_collection --> validate_updated_collection
    validate_updated_collection --> ingest_updated_collection
    ingest_updated_collection --> stac

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
