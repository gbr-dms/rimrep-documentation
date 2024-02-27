# Previous architecture diagrams

## DMS December 2023

Before Keycloak/KrakenD and Frictionless were incorporated into the system 

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
  classDef green fill:#ccffcc,stroke:#00ff00;

  subgraph "DMS"
    data_pipeline(Data Pipeline)
    data_api(Data API):::green
    metadata_api(Metadata API):::green
    data_storage[(Data Storage)]:::green
    catalog[(Catalog)]
  end

  data_providers((Data Providers))
  external_metadata(("Metadata Providers"))

  external_users((External Users))

  data_providers --> |Ingested by| data_pipeline
  external_metadata --> |Harvested by| data_pipeline
  data_pipeline -->|Configures| data_api
  catalog -->|Configure and publish to| metadata_api
  data_pipeline -->|Publish metadata to| catalog
  data_pipeline -->|Publish data to| data_storage
  data_storage -->|Access by| data_api
  data_api -->|Access by| external_users
  data_storage -->|"#quot;Direct#quot; access by"| external_users
  metadata_api -->|Access by| external_users

  metadata_api-. Points to .-> data_api
  metadata_api-. Points to .-> data_storage
```



## RIMReP DMS Phase 1

From [RIMReP DMS Phase 1 Report - Scoping Assessment (May 2022)](https://universitytasmania.sharepoint.com/sites/RIMRePDMS/Shared%20Documents/Forms/AllItems.aspx?ga=1&id=%2Fsites%2FRIMRePDMS%2FShared%20Documents%2FDocs%2FDMS%20%2D%20Phase1%2FRIMReP%5FDMS%2Dphase1%5FReport%5FFINAL%2Epdf&viewid=0566ab7b%2Def7e%2D4348%2Db059%2Dfff671d97956&parent=%2Fsites%2FRIMRePDMS%2FShared%20Documents%2FDocs%2FDMS%20%2D%20Phase1) talk to @diodon for access

<img width="900" alt="image" src="https://user-images.githubusercontent.com/6187649/211437208-89124d62-3702-4af9-8aa9-84a3723615eb.png">

## RIMReP DMS Phase 2 - Townsville meeting

From [Minutes RIMReP meeting in Townsville 4 August 2022](https://universitytasmania.sharepoint.com/sites/RIMRePDMS/Shared%20Documents/Forms/AllItems.aspx?ga=1&id=%2Fsites%2FRIMRePDMS%2FShared%20Documents%2FGeneral%2FMeeting%5Fnotes%2FMinutes%20RIMReP%20meeting%20in%20Townsville%204%20August%202022%2Epdf&viewid=0566ab7b%2Def7e%2D4348%2Db059%2Dfff671d97956&parent=%2Fsites%2FRIMRePDMS%2FShared%20Documents%2FGeneral%2FMeeting%5Fnotes) talk to @diodon for access

<img width="900" alt="image" src="https://user-images.githubusercontent.com/6187649/210675708-cef14102-3375-4a83-bd03-88f81fcd4522.png">

## Metadata flow

![Metadata flow](images/metadata-flow.png)
