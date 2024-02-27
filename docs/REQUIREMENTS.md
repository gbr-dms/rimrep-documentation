# RIMReP DMS Requirements

- [RIMReP DMS Requirements](#rimrep-dms-requirements)
  - [Documents/Reports](#documentsreports)
  - [General](#general)
  - [Data API](#data-api)
  - [Metadata API](#metadata-api)
  - [Metadata entry tool (MET)](#metadata-entry-tool-met)
  - [Data pipeline](#data-pipeline)
  - [Auth](#auth)
  - [Admin dashboard](#admin-dashboard)
  - [Documentation](#documentation)
  - [Transition Plan](#transition-plan)
  - [Out of scope](#out-of-scope)

## Documents/Reports

In order of high to low relevance

**Agreed upon requirements**:

- [Townsville meeting minutes (August 2022)](https://universitytasmania.sharepoint.com/sites/RIMRePDMS/Shared%20Documents/Forms/AllItems.aspx?ga=1&id=%2Fsites%2FRIMRePDMS%2FShared%20Documents%2FGeneral%2FMeeting%5Fnotes%2FMinutes%20RIMReP%20meeting%20in%20Townsville%204%20August%202022%2Epdf&viewid=0566ab7b%2Def7e%2D4348%2Db059%2Dfff671d97956&parent=%2Fsites%2FRIMRePDMS%2FShared%20Documents%2FGeneral%2FMeeting%5Fnotes) _Ask @diodon for access_
- [RIMReP DMS Phase 2 - Proposal Application (August 2022)](https://universitytasmania.sharepoint.com/:w:/r/sites/RIMRePDMS/_layouts/15/Doc.aspx?sourcedoc=%7BC7B6B18C-562F-4CA6-8794-C8DE70681046%7D&file=000-CURRENT_GBRF_DMS%20Phase%202%20Application%20Form_20220912.docx&action=default&mobileredirect=true) _Ask @diodon for access_

**Secondary documents/proposals/etc**:

- [RIMReP DMS Phase 1 Report - Scoping Assessment (May 2022)](https://universitytasmania.sharepoint.com/sites/RIMRePDMS/Shared%20Documents/Forms/AllItems.aspx?ga=1&id=%2Fsites%2FRIMRePDMS%2FShared%20Documents%2FDocs%2FDMS%20%2D%20Phase1%2FRIMReP%5FDMS%2Dphase1%5FReport%5FFINAL%2Epdf&viewid=0566ab7b%2Def7e%2D4348%2Db059%2Dfff671d97956&parent=%2Fsites%2FRIMRePDMS%2FShared%20Documents%2FDocs%2FDMS%20%2D%20Phase1) _Ask @diodon for access_
- [RIMReP DMS Project Plan (November 2022)](https://universitytasmania.sharepoint.com/sites/RIMRePDMS/Shared%20Documents/Forms/AllItems.aspx?ga=1&id=%2Fsites%2FRIMRePDMS%2FShared%20Documents%2FDocs%2FRIMReP%20Reports%2F4C%5FRIMREP%5FDMSImplementation%5FProject%20Summary%5F2022%2D11%2D31%2Epdf&viewid=0566ab7b%2Def7e%2D4348%2Db059%2Dfff671d97956&parent=%2Fsites%2FRIMRePDMS%2FShared%20Documents%2FDocs%2FRIMReP%20Reports) _Ask @diodon for access_
- [Data management strategy report (2018)](https://universitytasmania.sharepoint.com/sites/RIMRePDMS/Shared%20Documents/Forms/AllItems.aspx?ga=1&id=%2Fsites%2FRIMRePDMS%2FShared%20Documents%2FDocs%2FData%20management%20strategy%20report%20%282018%29%2Epdf&viewid=0566ab7b%2Def7e%2D4348%2Db059%2Dfff671d97956&parent=%2Fsites%2FRIMRePDMS%2FShared%20Documents%2FDocs) _Ask @diodon for access_
- [Data audit and data practices review report (2018).pdf](https://universitytasmania.sharepoint.com/sites/RIMRePDMS/Shared%20Documents/Forms/AllItems.aspx?ga=1&id=%2Fsites%2FRIMRePDMS%2FShared%20Documents%2FDocs%2FData%20audit%20and%20data%20practices%20review%20report%20%282018%29%2Epdf&viewid=0566ab7b%2Def7e%2D4348%2Db059%2Dfff671d97956&parent=%2Fsites%2FRIMRePDMS%2FShared%20Documents%2FDocs) _Ask @diodon for access_
- [Business Analyst Report (2019)](https://universitytasmania.sharepoint.com/sites/RIMRePDMS/Shared%20Documents/Forms/AllItems.aspx?ga=1&id=%2Fsites%2FRIMRePDMS%2FShared%20Documents%2FDocs%2FBusiness%20Analyst%20Report%20%282019%29%2Epdf&viewid=0566ab7b%2Def7e%2D4348%2Db059%2Dfff671d97956&parent=%2Fsites%2FRIMRePDMS%2FShared%20Documents%2FDocs) _Ask @diodon for access_

## General

- To use open standards APIs (eg OGC API for features)
- To follow OpenAPI spec
- To use FOSS and contribute back to projects
- Prioritise open access of data

## Data API

- Public access with auth for restricted datasets (See [Auth](#auth))
- Realtime API with pagination support
- Support common agreed upon formats (client facing - that is these formats are returned by Data API) - for example
  - Vector: GeoJSON, shapefile
  - Raster: Geotiff, NetCDF, CoverageJSON
  - Tabular: CSV, Parquet
- Subset data by spatial and/or temporal extent
  - This may be realtime/synchronous for smaller datasets
  - This may be delayed/asynchronous for larger datasets
- Web-based API documentation

Possible additions:

- Clip by region (admin area, GBR region, etc) or polygon
- Subset/filter data by variables (numeric or text-based) + possibly CQL or some other query language
- "perma-link" with DOI or some global unique identifier for data API queries (to be refined)
- Version control - most likely out-of-scope

## Metadata API

- Public access with auth for restricted datasets (See [Auth](#auth))
- Spatial/temporal extent
- Summary of variables/dimensions
- Links to data
  - Data API
  - Direct access
  - External data source/API
  - External metadata
- A minimal set of additional fields that are relevant to the DMS (i.e. for purposes of data discovery)
  - Date last updated
  - Links to documentation
  - Keyword list, maybe several keyword lists (e.g. theme KW, geography KW, use case KW)
  - Project metadata (summary, contact persons, methodology, references, etc)
- Standard vocabularies (if applicable) eg to describe the variables, units, measurement methods, etc
- Web front-end (either develop or support the development of). This should be as simple as possible

## Metadata entry tool (MET)

- Self-service web front-end
- Supports vocabularies
- Temporary tool for initial metadata entry and data provider engagement
  - It will only exist for this phase of the project (ending June 2024)
  - All metadata collected with the tool will be stored in an external metadata management system by the end of this phase of the project
- Non-essential metadata (for the purposes of DMS), will not be for public use — it will only be for ingestion into the external metadata management system
- It will only be used by data providers that have no metadata stored in external metadata management systems
- It will not be used for sensitive metadata entry
- It will not be included in security reviews or penetration testing
- It will be isolated from the rest of DMS - i.e. no tight integration with auth, data pipeline, metadata api

## Data pipeline

Definitely:

- Two types of workflows:
  - Active: we acquire, transform and store data
  - Reactive: we are sent data, we transform and store data
- Active workflows may be scheduled
- We may want to cache data that we retrieve in its original format
- When data is updated, we should create or update STAC metadata
- When a workflow fails, we should be notified
- All data is transformed into a few ARCO data formats with common structure
- APIs are updated with required metadata

Unknowns:

- Harvest metadata/data from external sources (eg THREDDs, FTP, ...)

## Auth

- Both data and metadata API will be public, but authenticated users can access restricted data/metadata
- Authentication may or may not be a function of DMS
- Continuous system monitoring for near real-time security information
- Users will be authenticated using standard protocols, and the authenticated system will be managed internally or by a relevant organisation

## Admin dashboard

This is pretty vague

> A fit-for-purpose administration dashboard will be agreed on and delivered, which can be used
> for functions such as user management, data management, metadata management or other
> functions as required

Possibilities:

- User management
  - Add/remove users
  - Grant access
  - ...
- Metadata management
  - Create/edit metadata
  - Ties in "self-service metadata entry tool"
- Monitoring for data pipeline
- Usage metrics

## Documentation

Detailed specifications and system design will be delivered that include:

- A clear vision for the system
- A prioritised list of use cases
- Statutory and regulatory requirements
- A prioritised list of target datasets and requirements for ingested data
- Software and data licensing
- High-level architecture
- Technical risks
- Detailed work breakdown structure and delivery schedule.

A plan will be made to support the system to be transitioned into the hands of an entity that can
ensure the system's ongoing governance, maintenance and operations.

- Documentation will be maintained and updated to an agreed standard.

## Transition Plan

A plan will be made to support the system to be transitioned into the hands of an entity that can
ensure the system's ongoing governance, maintenance and operations.

- Documentation will be maintained and updated to an agreed standard.
- The receiving entity will need to demonstrate technical capability to operate the DMS as well as commitment to the RIMReP project

The plan will consist of

- Code repositories documented according to common practices
- An estimated cost of maintenance
- Provisions required for transferring the cloud infrastructure to the new owner

## Out of scope

- Creating aggregated or derived products. This belongs outside the DMS in the analytics platform
- Harmonisation of ontologies and conceptual models. This would need to happen in an analytics platform or be negotiated with the data provider
- The DMS will not cache very large datasets or datasets that are already well managed and accessible. It can support metadata entries that include links to documentation and API endpoints as appropriate
- Data curation, cleansing and extensive quality assurance. If a dataset is not at a sufficient level of quality it won’t be published
- A metadata management system. The DMS will not be the point-of-truth for metadata records. If external metadata records are not available, we may use the MET to create temporary records that will be migrated to an external metadata management system by the end of this phase of the project.
