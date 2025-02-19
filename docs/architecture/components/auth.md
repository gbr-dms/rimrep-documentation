# Auth

## Summary


This document presents the Architecture for Authentication and Authorization flow for the following services:


- [`rimrep-pygeoapi`](https://github.com/gbr-dms/rimrep-pygeoapi) Data API
- [`rimrep-stac-fastapi`](https://github.com/gbr-dms/rimrep-stac-fastapi) Metadata API
- [`rimrep-stac-browser`](https://github.com/gbr-dms/rimrep-stac-browser) Metadata API Web Client
- S3-public buckets
- S3-private buckets

Additionally, it describes the sequence of interactions in two flows:

Machine-to-Machine (M2M) Flow: This flow demonstrates how a Machine (Client) can obtain an access token using the Client Credentials Grant and utilize it to call APIs via the Krakend API Gateway.

User Browser Flow: This flow showcases how a user's browser interacts with the Krakend API Gateway for authentication and authorization purposes.

The document provides insights into the steps and interactions involved in the mentioned flows, enabling a clear understanding of how the services handle authentication and authorization.


## Architecture 

```mermaid
%%{
  init: {
    'theme': 'base',
    'themeVariables': {
      'edgeLabelBackground': '#ffffff',
      'tertiaryTextColor': '#0f00aa',
      'clusterBkg': '#FFFF00',  // Change to yellow (#FFFF00)
      'clusterBorder': '#0f00aa'
    }
  }
}%%





flowchart TB

classDef redUnfinished fill:#ffeeee,stroke:#ff0000,stroke-dasharray: 5 5
classDef red fill:#ffcccc,stroke:#ff0000;
classDef purple fill:#ebccff,stroke:#9900ff;
classDef green fill:#ccffcc,stroke:#00ff00;
classDef orange fill:#ffddb3,stroke:#ff8000;
classDef blue fill:#cce0ff,stroke:#0000ff;
classDef teal fill:#ccffdd,stroke:#00cc99;
classDef cyan fill:#ccffff,stroke:#009999;
classDef grey fill:#cccccc,stroke:#666666;
classDef dark_grey fill:#666666,stroke:#000000;
classDef purpleUnfinished fill:#ebccff,stroke:#9900ff,stroke-dasharray: 5 5

User(user: fa:fa-users):::grey 
App/Machine(App/Machine fa:fa-laptop-code):::green 

subgraph "k8s-EKS"
  metadata_api(stac-browser):::red
  data_api(pygeoapi):::red
  API_Gateway(KrakenD API Gateway fa:fa-sitemap):::redUnfinished
  Keycloak(Keycloak IDP fa:fa-user-lock):::purple
  aws-signature-plugin([KrakenD S3 Plugin fa:fa-plug]):::redUnfinished


end

  subgraph legend 
  direction TB
  UserFlow("User Browser Flow": fa:fa-users):::orange 
  M2MFlow("M2M Flow": fa:fa-laptop-code):::blue 
  KrakenDFlow("KrakenD Flow": fa:fa-sitemap):::dark_grey 

end


subgraph AWS
  direction LR
  s3-public-bucket(s3-public fa:fa-folder):::red 
  s3-private-bucket(s3-private fa:fa-folder):::red

  keycloakDB[(keycloakDB)]:::purple
  MetadataDB[(MetadataDB)]:::teal
  s3-public-bucket ~~~ s3-private-bucket
  s3-private-bucket ~~~ keycloakDB
end

User -- 1.User accessing through browser --> data_api
User -- 1.User accessing through browser --> metadata_api
data_api -- 2.App to Redirect to Keycloak login --> Keycloak
metadata_api -- 2.App to Redirect to Keycloak login --> Keycloak
Keycloak -- 3.User redirected to pygeo api upon login --> data_api
Keycloak -- 3.User redirected to stac browser upon login --> metadata_api
App/Machine -- 1.Client Credentials flow --> Keycloak
App/Machine -- 2.Access API/S3 Bucket with Authorisation token --> API_Gateway

Keycloak -- Connect to keycloakDB--> keycloakDB
metadata_api -- Connect to Metadata DB--> MetadataDB

data_api --> API_Gateway --> s3-public-bucket
data_api --> API_Gateway --> s3-private-bucket
API_Gateway --> aws-signature-plugin
aws-signature-plugin --> s3-public-bucket
aws-signature-plugin --> s3-private-bucket
  s3-public-bucket ~~~ s3-private-bucket
  s3-private-bucket ~~~ keycloakDB

linkStyle 2 stroke:orange
linkStyle 3 stroke:orange
linkStyle 4 stroke:orange
linkStyle 5 stroke:orange
linkStyle 6 stroke:orange
linkStyle 7 stroke:orange
linkStyle 8 stroke:blue
linkStyle 9 stroke:blue
linkStyle 10 stroke:Purple
linkStyle 11 stroke:Red

```
### Machine to Machine Flow

```mermaid
%%{
  init: {
    'theme': 'base',
    'themeVariables': {
      'edgeLabelBackground': '#ffffff',
      'tertiaryTextColor': '#0f00aa',
      'clusterBkg': '#FFFF00',  // Change to yellow (#FFFF00)
      'clusterBorder': '#0f00aa'
    }
  }
}%%
%%{
  init: {
    'theme': 'base',
    'themeVariables': {
      'edgeLabelBackground': '#ffffff',
      'tertiaryTextColor': '#0f00aa',
      'clusterBkg': '#FFFF00',  // Change to yellow (#FFFF00)
      'clusterBorder': '#0f00aa'
    }
  }
}%%
sequenceDiagram
    participant Machine as "Machine (Client)" #FFA500
    participant Krakend as "Krakend API Gateway" #FFD700
    participant Keycloak as "Keycloak IDP" #1E90FF
    participant STAC_API as "Stac-Browser/Pygeoapi" #00FF7F
    participant Custom_Plugin as "Krakend S3 Plugin" #FF1493
    participant S3_Public_Bucket as "S3 Public Bucket" #FF6347
    participant S3_Private_Bucket as "S3 Private Bucket" #BA55D3

    Machine->>Keycloak: Request Access Token using Client Credentials
    Keycloak->>Machine: Return Access Token

    Note over Machine, Keycloak: Machine gets the Access Token

    Machine->>Krakend: Call API (stac-browser/pygeoapi) with Access Token
    Krakend->>Krakend: Validate the authenticity of the token with Keycloak IDP
    Krakend->>Krakend: Verify User Role in token

    Krakend->>STAC_API: Proxy Request to stac-browser/pygeoapi with Access Token
    STAC_API->>Keycloak:Request Access Token using Client Credentials
    Keycloak->>STAC_API:Return Access Token

    STAC_API->> Krakend: Call API (S3 buckets via KrakenD Gateway) with Access Token ,[Follow the Machine accessing S3 buckets]
    Krakend->> STAC_API: Receive Data from S3 buckets
    STAC_API->> STAC_API: Process Data
    STAC_API->>Krakend: Return Data

    Krakend->>Machine: Receive Data from stac-browser/pygeoapi

    Note over Machine, Krakend: Machine accesses stac-browser/pygeoapi

    Machine->>Krakend: Call API (S3 public buckets) with Access Token
    Krakend->>Custom_Plugin: Proxy Request to Krakend S3 Plugin  with Access Token
    Custom_Plugin ->> Custom_Plugin : Uses the AWS credentials to trigger AWS S3 API  
    Custom_Plugin->>S3_Public_Bucket: Request Data from S3 Public Bucket
    S3_Public_Bucket->>Custom_Plugin: Return Data
    Custom_Plugin->>Krakend: Forward Data
    Krakend->>Machine: Receive Data from S3_Public_Bucket

    Note over Machine, Krakend: Machine accesses S3 Public Bucket

    Machine->>Krakend: Call API (S3 private buckets) with Access Token
    Krakend->>Krakend: check the authorisation on the Dataset
    Krakend->>Custom_Plugin: Proxy Request to Krakend S3 Plugin  with Access Token
    Custom_Plugin->>S3_Private_Bucket: Request Data from S3 Private Bucket
    S3_Private_Bucket->>Custom_Plugin: Return Data
    Custom_Plugin->>Krakend: Forward Data
    Krakend->>Machine: Receive Data from S3_Private_Bucket

    Note over Machine, Krakend: Machine accesses S3 Private Bucket



```
### User Flow

```mermaid
%%{
  init: {
    'theme': 'base',
    'themeVariables': {
      'edgeLabelBackground': '#ffffff',
      'tertiaryTextColor': '#0f00aa',
      'clusterBkg': '#FFFF00',  // Change to yellow (#FFFF00)
      'clusterBorder': '#0f00aa'
    }
  }
}%%
sequenceDiagram
    participant User
    participant FrontendApp as "Stac-Browser/Pygeoapi"
    participant Keycloak as "keycloak IDP (via OAuth2-Proxy)"
    participant Krakend as "Krakend API Gateway" #FFD700

    User->>FrontendApp: Access Via Brower

    FrontendApp->>Keycloak: Redirect to Keycloak Login page

    Note over FrontendApp, Keycloak: User logs in

    Keycloak->>+FrontendApp: Redirect with Access Token by Creating a User Session

    Note over FrontendApp, Keycloak: User Session

    
        alt S3 Bucket Flow: Pygeo api accessing S3 buckets via KrakenD API Gateway
        FrontendApp->>Krakend: Call S3 buckets using KrakenD API Gateway [Follow the Machine accessing S3 buckets flow]
        Krakend->>FrontendApp: Return Data from S3 buckets
    end

   
   

    FrontendApp->>User: Serve Both Private/Public Datasets Collections

    User->>FrontendApp: Click "Private DataSet"
    FrontendApp->>Krakend: Make a Call to Krakend Gateway
    Krakend->>Keycloak: Validate the user Authorisation(Via Roles)
    Note over Krakend, FrontendApp: User is Authorised
    Krakend->>FrontendApp: User is served with Private Dataset Details
    Note over Krakend, FrontendApp: User is Not Authorised
    Krakend->>FrontendApp: User is served with 403 Unauthorised


    User->>FrontendApp: Click "Logout"
    FrontendApp->>Keycloak: Redirect to Keycloak logout
    Keycloak->>FrontendApp: Redirect to Frontend logout
    FrontendApp->>+FrontendApp: Clear Session

    FrontendApp->>User: Clear Session
    Note over FrontendApp, Keycloak: User Logout

    alt Failure: User accesses FrontendApp URLs(private datasets) directly
        User->>FrontendApp: Access FrontendApp URL
        FrontendApp->>Keycloak: Redirect to Keycloak login page and follow the main flow
    end




```
### Keycloak

Keycloak (https://www.keycloak.org/) is a self-hosted and open-source identity provider (IDP) offering an excellent alternative to Auth0. It boasts fine-grained authorisation capabilities at various levels, allowing for precise access control. Role-based access control (RBAC) is supported for machine-to-machine (M2M) authorisation, granting granular permissions. As a widely adopted enterprise solution in the industry, Keycloak efficiently manages all users and their respective roles within the platform.


### KrakenD API Gateway

KrakenD (https://www.krakend.io/) is an open-source and flexible solution. Seamlessly integrating with cloud-native infrastructure, KrakenD can be easily deployed using an operator or Helm. Setting it apart from other popular choices like Kong or Tyk, KrakenD operates in a stateless manner, ensuring smooth scalability and reliability. With a wide range of features to optimize API management, such as rate limiting and robust request and response transformations, KrakenD empowers developers to deliver top-notch performance.

Furthermore, KrakenD simplifies authentication and authorization implementation by seamlessly integrating with OIDC providers like Auth0 or Keycloak, streamlining API protection. By connecting with Keycloak, KrakenD validates the Authorization token, and its configuration allows RBAC on the URLs (datasets) to apply filtering based on Keycloak roles. This enables precise control over access, providing enhanced security and functionality for your APIs.

### Krakend S3 Plugin

- [`rimrep-krakend-plugin`](https://github.com/gbr-dms/rimrep-krakend-plugin)  The KrakenD S3 Plug-in, written in GO, is seamlessly integrated into the base image of KrakenD. This powerful plug-in utilises AWS client credentials to directly interact with S3 buckets using the AWS S3 API, eliminating the need for any S3 proxy.

By combining the KrakenD S3 Plug-in with the KrakenD API Gateway, you can efficiently serve S3 bucket data with precise and fine-grained authorisation at the dataset level. This integration allows for secure and controlled access to specific datasets, ensuring enhanced security and data management for your S3 resources.



