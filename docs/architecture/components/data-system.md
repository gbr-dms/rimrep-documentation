# Data system

See [Data API requirements](../../requirements.md#data-api)

## Summary

- Authentication [`oauth2proxy`](https://oauth2-proxy.github.io/oauth2-proxy/) (See [auth architecture](auth.md))
- Data API backend (and simple frontend): [`rimrep-pygeoapi`](https://github.com/aodn/rimrep-pygeoapi)
- Data API authorization: [`s3proxy`](https://oxyno-zeta.github.io/s3-proxy/)
- Direct access: AWS S3 or [`s3proxy`](https://oxyno-zeta.github.io/s3-proxy/)

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

  subgraph "AWS S3"
    public_data_storage[(Public data)]
    private_data_storage[(Private data)]
  end

  subgraph "k8s"

    data_api(Pygeoapi)
    s3_proxy(S3 Proxy):::red
    oauth2_proxy(Oauth2 Proxy):::red
  end

  external_users((External Users))

  public_data_storage -->|S3/HTTP REST| data_api & external_users
  data_api -->|OGC APIs| oauth2_proxy

  private_data_storage -->|S3| s3_proxy
  s3_proxy -->|HTTP REST| data_api & oauth2_proxy

  oauth2_proxy --> external_users
```

### Data storage

All data is stored in AWS S3 in two buckets

- `rimrep-data-public` for public data
- `rimrep-data-limited-access` for limited access data
- See [data sensitivity classification](../../data/data-sensitivity.md) for more details

We support two data formats:

- zarr for multi-dimensional gridded numerical data
- parquet for tabular/vector data

### Data API

We are using [`rimrep-pygeoapi`](https://github.com/aodn/rimrep-pygeoapi) (a fork of [`pygeoapi`](https://github.com/geopython/pygeoapi/)) to publish OGC APIs for data access.

This will publish [OGC API Features](https://ogcapi.ogc.org/features/) (from geo/parquet) and [OGC API Coverages](https://ogcapi.ogc.org/coverages/) (from zarr).

## Auth

See [Auth architecture](auth.md) documentation for more detailed information.

Currently, `pygeoapi` requires authentication for all datasets. In the future, we will make all datasets open access by default, and only require authorization for limited access datasets.

Public data is in the `rimrep-data-public` S3 bucket and is accessible to all users without authentication (directly to AWS S3). Limited access data is in the `rimrep-data-limited-access` S3 bucket and is only accessible to authorized users through [`s3proxy`](https://oxyno-zeta.github.io/s3-proxy/). This applies to both "direct access" and "data API" access.

- Direct access: [`s3proxy`](https://oxyno-zeta.github.io/s3-proxy/) is used to proxy requests to the private S3 bucket. It doesn't support S3 API, but does support HTTP REST API.
- Data API access: `pygeoapi` does not access to the `rimrep-data-limited-access` bucket directly (through S3), and it does not have authorization built in. When a user makes a request, the `Authorization` token is passed through to [`s3proxy`](https://oxyno-zeta.github.io/s3-proxy/) which authorizes the request.

Data API authorization is handled by [`s3proxy`](https://oxyno-zeta.github.io/s3-proxy/), currently this is hard-coded configuration. In the future we will use [Open Policy Agent (OPA)](https://www.openpolicyagent.org/) to provide fine-grained authorization.
