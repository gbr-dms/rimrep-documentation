# Data Ingestion DoD

Definition of done for data ingestion.

- Dataset satisfies "Data ingestion guidelines"
- Argo Workflow created to convert and publish dataset
  - Chron workflow set up with appropriate frequency for regularly-updated data
- If any of these apply, documentation created to describe:
  - Manual data download steps
    - Input is stored in `gbr-dms-data-shared-access` bucket
  - Additional manual steps needed to run Argo Workflow
  - Non-standard data license
- For limited access data:
  - Appropriate access checks set up on KrakenD
