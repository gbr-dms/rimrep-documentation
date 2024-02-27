# Metadata Ingestion Guidelines

WIP

See [Metadata System architecture](../architecture/components/metadata-system.md)

Three types of metadata

- Metadata record exists (external to DMS).
- Metadata record is to be created through MET (for purposes of ingestion in to an external metadata management system).
- No metadata record exists, but dataset is well-known and documented, and "implicit" metadata exists.
  - For example ABS census data

## Standard lists/vocabs

As per [REQUIREMENTS.md#metadata-api](https://github.com/aodn/rimrep-dms/blob/main/docs/REQUIREMENTS.md#metadata-api), we want to have standard vocabs/enumerated values for data variables, keywords, etc.

- The vast majority of - if not all - datasets will require us to manually match data variables with our standard lists/vocabs.
- Some data providers may be happy to engage with us to either provide metadata compliant with our standard/lists - or at very least verify the metadata manually created by us.
- Either way, I think the easiest way forward is for us to do a first pass of manually creating the required metadata when we get access to the data.
- The MET will not be used for this type of data entry for metadata records that exist externally.

We will need to perform validation on our manually curated metadata as the data is harvested:

- If the data structure changes, we need to be notified so we can update metadata.

## Metadata record exists (external to DMS)

We aim to automatically harvest **all** external metadata records automatically.

Depending on data/metadata update frequency, we can schedule harvesting on different time-scales (eg daily, weekly, ...).

If there are metadata records with particularly poor access (eg externally hosted but requires human intervention for access), and we do not expect the metadata to change, we can manually ingest the metadata record. In this case, we must document how and why we ingested the metadata record.

### Standard lists/vocabs from external records

- We may be able to harvest and validate keywords/variables against our standard lists/vocabs.
- Most likely, we will need to manually match data variables/keywords with our standard lists/vocabs

## Metadata record is to be created through MET

Following https://github.com/aodn/rimrep-dms/blob/main/docs/REQUIREMENTS.md#metadata-entry-tool-met, the MET will be used by data providers to create a new metadata record **only when all of the following are true**:

- A record does not exist in an external metadata management systems
- The data provider does not have a metadata management system
- The data provider **wants** to create a metadata record
- The data provider agrees to the new metadata record being ingested into an external metadata management system by the end of the project

After the Metadata record has been created, we temporarily use the generated XML file until it has been ingested into an external metadata management system.

When the metadata record is in an external metadata management system, we will transition to automatically harvesting the record.

Currently, we are assuming that said external metadata management system is GBRMPA's Reef Knowledge System GeoNetwork instance.

### Standard lists/vocabs from MET output

- We can _temporarily_ use the output JSON file to generate STAC fields.
- When the record is ingested into an external metadata management system, it will be treated like all other external metadata records.

## No metadata record exists, but "implicit" metadata exists

In this case, basically, we create metadata ourselves.

### Standard lists/vocabs from implicit metadata

We have to manually match data variables/keywords with our standard lists/vocabs
