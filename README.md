# RIMReP Data Management System

This serves as a main/home repo for all things related to RIMReP DMS.

## Docs

- [All docs](docs/README.md)
- [Requirements](docs/REQUIREMENTS.md)
- [Architecture overview](./docs/architecture/README.md)

## Planning

- [Roadmap](docs/ROADMAP.md)
- [DMS Planning project](https://github.com/orgs/aodn/projects/25)
- [DMS Datasets project](https://github.com/orgs/aodn/projects/29)

## Components

- [`rimrep-pygeoapi`](https://github.com/aodn/rimrep-pygeoapi) Data API
- [`rimrep-stac-fastapi`](https://github.com/aodn/rimrep-stac-fastapi) Metadata API
- [`rimrep-stac-browser`](https://github.com/aodn/rimrep-stac-browser) Metadata API Web Client
- [`rimrep-metcalf`](https://github.com/aodn/rimrep-metcalf/) Metadata Entry Tool (MET)
- [`rimrep-dashboard`](https://github.com/aodn/rimrep-dashboard) Dashboard Web Client
- [`rimrep-catalog`](https://github.com/aodn/rimrep-catalog) Catalog - single source of truth
- [`rimrep-data-pipeline`](https://github.com/aodn/rimrep-data-pipeline) Data Pipeline
- [`rimrep-terraform`](https://github.com/aodn/rimrep-terraform) Terraform setup
- [`rimrep-flux`](https://github.com/aodn/rimrep-flux) Flux and Kustomize k8 management
- [`rimrep-examples`](https://github.com/aodn/rimrep-examples) RIMReP examples and documentation
- [`rimrep-examples-private`](https://github.com/aodn/rimrep-examples-private) A private version of `rimrep-examples`
- [`rimrep-training`](https://github.com/aodn/rimrep-training) Specific examples to be used for training days
- [`rimrep-argo-workflow`](https://github.com/aodn/rimrep-argo-workflow) Argo Workflows for data pipeline and metadata pipeline
- [`gbr-dms-e2e-tests`](https://github.com/aodn/gbr-dms-e2e-tests) End-to-end tests for the whole system

## Using SSO to access AWS

Use [this link to sign on to AWS](https://sso.aodn.org.au/home/amazon_aws_sso/0oa7nsao8eP5aU9YM5d7/aln1ghfn5xxV7ZPbE1d8).
If you don't have access, send a request to Eduardo Klein to be added.

To use the command line to access AWS do the following:

1. Install the AWS CLI app
2. At the command line, type `aws configure sso`
3. Name the SSO session something common, like `imos` or `rimrep`
4. Use this as the URL: `https://d-976763a7b2.awsapps.com/start`
5. Region should be `ap-southeast-2`
6. Registration scopes can be left as default.
