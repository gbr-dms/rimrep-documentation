# Data Ingestion DoD

Definition of done for data ingestion.

Very basic for open-access data only.

- Dataset satisfies "Data ingestion guidelines"
- Argo Workflow created to convert dataset with documentation for the following
  - Name
  - Data update frequency
  - Manual data download steps if applicable
    - Input is stored in `rimrep-data-input` bucket
  - Instructions on how to run Argo Workflow (if one-off)
  - Data license
- For non-predefined geometries
  - zarr/parquet stored in `rimrep-data-public` bucket
  - Collection is added to `pygeoapi` configuration
- For predefined geometries
  - STAC collection is created for each dataset
  - STAC items are created for each region/geometry
