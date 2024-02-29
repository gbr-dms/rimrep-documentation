# RIMReP Data Management System

IMOS is leading a project to deliver a fit-for-purpose Data Management System (DMS) for the Reef 2050 Integrated Monitoring and Reporting Program (RIMReP).

The DMS will be critical infrastructure that underpins the Program to create a better understanding of what’s happening on the Great Barrier Reef. Also, the DMS will be a critical service in support of RIMReP’s Reef Knowledge System (https://reefknowledgesystem.gbrmpa.gov.au/).

The DMS architecture is conceived as a scalable data-agnostic system based on a cloud-native environment and built using open-source tools based on standards. The system will collect and update data and metadata from data providers in an automated way, transform the data collections into analysis-ready cloud-optimised (ARCO) formats, and provide the end user with a delivery mechanism through dedicated Application Programming Interface (API) services or direct connection to the cloud storage.

For more details go to the project webpage or contact us at info-dms at utas dot edu dot au

## Access the DMS catalogue and data services

The DMS catalogue is available at http://stac.reefdata.io/browser. If your organisation is an [AAF partner](https://aaf.edu.au/subscribers/) you can use your email credentials to log in. If not, you can write to info.dms@utas.edu.au and we will provide your access credentials


## Docs

- [All docs](docs/README.md)
- [Requirements](docs/REQUIREMENTS.md)
- [Architecture overview](./docs/architecture/README.md)


## Components


### Private structural repositories

- <a href="https://github.com/aodn/rimrep-pygeoapi" style="color: red; text-decoration: underline;text-decoration-style: dotted;">rimrep-pygeoapi</a> Data API
- <a href="https://github.com/aodn/rimrep-stac-fastapi" style="color:red;text-decoration: underline;text-decoration-style: dotted;">rimrep-stac-fastapi</a> Metadata API
- <a href="https://github.com/aodn/rimrep-stac-browser" style="color:red;text-decoration: underline;text-decoration-style: dotted;">rimrep-stac-browser</a> Metadata API web client 
- <a href="https://github.com/aodn/rimrep-metcalf" style="color:red;text-decoration: underline;text-decoration-style: dotted;">rimrep-metcalf)</a> Metadata Entry Tool (MET)
- <a href="https://github.com/aodn/rimrep-dashboard" style="color:red;text-decoration: underline;text-decoration-style: dotted;">rimrep-dashboard</a> Dashboard Web Client
- <a href="https://github.com/aodn/rimrep-catalog" style="color:red;text-decoration: underline;text-decoration-style: dotted;">rimrep-catalog</a> Catalog - single source of truth
- <a href="https://github.com/aodn/rimrep-data-pipeline" style="color:red;text-decoration: underline;text-decoration-style: dotted;">rimrep-data-pipeline</a> Data Pipeline
- <a href="https://github.com/aodn/rimrep-terraform" style="color:red;text-decoration: underline;text-decoration-style: dotted;">rimrep-terraform</a> Terraform setup
- <a href="https://github.com/aodn/rimrep-flux" style="color:red;text-decoration: underline;text-decoration-style: dotted;">rimrep-flux</a> Flux and Kustomize k8 management
- <a href="https://github.com/aodn/rimrep-argo-workflow" style="color:red;text-decoration: underline;text-decoration-style: dotted;">rimrep-argo-workflow</a> Argo Workflows for data pipeline and metadata pipeline
- <a href="https://github.com/aodn/gbr-dms-e2e-tests" style="color:red;text-decoration: underline;text-decoration-style: dotted;">gbr-dms-e2e-tests</a> End-to-end tests for the whole system
- <a href="https://github.com/aodn/rimrep-examples-private" style="color:red;text-decoration: underline;text-decoration-style: dotted;">rimrep-examples-private</a> A private version of `rimrep-examples`
- <a href="https://github.com/aodn/rimrep-Rpackage" style="color:red;text-decoration: underline;text-decoration-style: dotted;">rimrep-Rpackage</a> an R package with useful functions to work with DMS data and metadata services (WiP)


### Public repositories

- [rimrep-examples](https://github.com/aodn/rimrep-examples) RIMReP examples and documentation
- [rimrep-training](https://github.com/aodn/rimrep-training) Specific examples to be used for training activities
- [rimrep-documentation](https://github.com/aodn/rimrep-documentation) This repository


