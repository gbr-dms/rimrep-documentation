# Argo Workflow Templates

This page describes the Argo Workflow templates used in the data ingestion workflows.
These templates have been created to be as reusable as possible, allowing to recombine
the same templates across different workflows to ingest different datasets.

## Template life-cycle

New workflows should always try to reuse these templates.
If new code needs to be created to ingest a new dataset, the following
steps should be followed in order to balance the need for reusability and
the need for fast development:

1. Write and test the code locally as much as possible.
2. Include the code directly in the workflow YAML file as a `script` template.
3. If the code could be reused in other workflows, refactor it into a new template in
   one of the files in the `tools` directory and add its description to this document.
4. If the template is used in multiple workflows, and/or is complex enough to benefit
   from unit testing and better packaging, move its code to the `rimrep-data-pipeline`
   repository, turn it into a command-line script, write tests for it, and use the
   new docker image that gets built from that code in the `tools` template where
   the code was originally.

More complex or core templates may skip directly to step 4.

## Tools currently available

### CSV

Templates to process CSV files.

- `split-by-column`: Splits a CSV file into multiple files based on the values of a column. Input/output via s3 paths (should probably be refactored to use artifacts instead).
- `pivot`: an interface to the [`pandas.DataFrame.pivot`](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.pivot.html) function, to reshape data from long to wide format.
- `formatting-data-type`: Reformats a csv file to ensure each column has the type specified in the give tableschema file.
- `add-lat-lon-columns`: Adds latitude and longitude columns with fixed values to a CSV file.
- `concat-csv`: Concatenates multiple CSV files into a single one.
- `des-qspatial-source-downloader`: Downloads data from the DES QSpatial API.
- `ala-source-downloader`: Downloads data from the ALA API.
- `obis-source-downloader`: Downloads data from the OBIS API.
- `obis-metadata-parser`: Parses metadata downloaded from OBIS API.

### Finalise

Hosts the `finalise` template, which is the last step in most workflows. It validates the data, generates metadata files for all of our services, and uploads the data to our S3 bucket (if needed).

- `finalise`: to be called at the end of every ingestion workflow. The `remote` input can be set to "true" if the data is already in S3, otherwise an artifact `data` should be provided with the data to be uploaded. `coordinates` should be set to a json dictionary with the names of the `x,y,z,t,geometry` columns/variables in the data (only include the ones that are present in the data). `dataset_id` and `format` should be set at workflow level for consistency but can also be passed directly here. Other parameters are optional and usually not needed.
- All other templates in this file are called by `finalise` and should not be called directly.

### FTP

Templates to download files from FTP servers.

- `get-latest-file`: gets the name of the latest file with the required extension in an FTP directory. Also returns "Update"/"NoUpdate" as a separate value to indicate if a file was found.
- `ftp-update-controller`: same as `get-latest-file` but also checks if that files already exists in a cache folder on S3.

### Github

Templates to interact with Github repositories, particularly our metadata in `rimrep-catalog`.

- `get-catalog-files`: gets our metadata catalog files. Can specify endpoints for jsonnet libraries to generate configuration json files (used by finalise to generate the right urls for different environments).
- `get-tableschema`: gets the tableschema file for a given dataset from our metadata catalog.
- `generate-datapkg-from-github-issue`: generates a datapackage.json file from a Github issue given the issue number.
- `upload-datapkg-to-github`: uploads a new datapackage.json file to our metadata catalog.
- `create-pr-in-catalog-repo`: creates a PR between two given branches in our metadata catalog.
- `generate-datapkg-and-upload`: combines `generate-datapkg-from-github-issue` and `upload-datapkg-to-github` to automatically generate a datapackage.json file from a Github issue and upload it to our metadata catalog.

### Harvester

Templates to harvest metadata from different sources. Used by the templates in the `metadata-harvest` directory.

- None of these templates should be called directly. They are called by the `metadata-harvest` templates and ultimately triggered by the `finalise` template.

### JSON

Templates to process JSON files.

- `grep-field-from-json`: extracts a field from a JSON file.

### NetCDF

Templates to process NetCDF files.

- `download-and-clip`: takes a list of urls in an artifact as input and downloads the files at those urls. It then clips the files to a given bounding box and saves them to a given s3 bucket and key.
- `download-netcdf`: same as `download-and-clip` but without clipping.
- `exclude-no-content-urls`: filters a list of urls to download with a list of urls to skip. Also computes the number of batches when the urls are split into groups of 100.
- `get-valid-partition-size`: Finds a partitioning size for converting groups of netcdf files into zarr that works with the desired chunk size.

### Notifications

Templates to send notifications to external services.

- `exit-handler-slack`: sends a message to our slack webhook with the status of the workflow calling this template. Usually used as an `onExit` template, with a `when` condition to only send the message when the workflow did not succeed.
- `notify-new-dataset-slack`: sends a message to our slack webhook to notify about a new issue created by our metadata harvester.

### Parquet

Templates to process Parquet files. Several of these templates accept multiple options as input to alter their behaviour. Check the comments in the templates' definitions for more information.

- `combine-metadata`: generates a `_metadata` file for a partitioned parquet dataset on S3.
- `geopandas-to-parquet`: converts a file that can be read by geopandas to geoparquet.
- `geodatabase-to-parquet`: converts a gdb file to geoparquet.
- `csv-to-parquet`: converts a csv file to parquet.
- `csv-to-geoparquet`: converts a csv file with columns for latitude and longitude to geoparquet.
- `json-to-parquet`: converts a json file to parquet.
- `json-to-geoparquet`: converts a json file with columns for latitude and longitude to geoparquet.
- `add-geoms`: joins a parquet file with a geoparquet file on a common column adding the geometries from the second file to the first.
- `add-column-metadata`: adds metadata to the columns of a parquet file; input taken as a second parquet file with column names as the index and metadata fields as other columns.
- `unzip-source-file`: unzips an artifact and returns a new artifact with the unzipped files. (not really restricted to parquet)
- `wrap-data`: takes a parquet file as an artifact and returns a new artifact with a single folder containing that file. Useful because we always expect parquet datasets to be folders with their partitions inside.
- `clip-data`: clips a geoparquet file to a given bounding box.

### S3

Templates to interact with S3.

- `path-exists`: checks if a given s3 url exists. Prints "true" or "false" as output.
- `list-objects`: lists all the objects with a given prefix on s3. Output is in the "existing-files" artifact.
- `url-upload`: downloads a file from an http url and uploads it to s3.
- `url-unzip`: downloads a zip file from an http url, unzips it, and uploads the unzipped files to s3. Expects the zip to contain only a single file.
- `file-unzip`: same as `url-unzip` but input/output is with artifacts rather than urls.
- `folder-upload`: uploads a folder to s3.
- `folder-empty`: deletes all the files in a folder (prefix) on s3.
- `file-upload`: uploads a file to s3.
- `generate-s3-uris`: generates a list of s3 uris from a list of http urls by using a given prefix and attaching as suffix the final part of the urls.
- `list-objects-uri`: like `list-objects` but returns the full s3 uri of each object rather than just the name.
- `download-and-clip-multi-nasa-netcdf`: downloads and clips multiple NASA Earthdata netcdf files, converts them to zarr and uploads them to s3.
- `generate-download-urls-and-uris`: generates a list of download urls for a thredds or ftp server and corresponding s3 urls for our cache bucket.


### STAC

Templates to interact with our STAC catalog. Not needed for normal data ingestion workflows as the `finalise` step takes care of generating the STAC metadata using these templates.

- `check-stac-collection-exist`: checks if a collection-id exists in our STAC catalog.
- `create-stac-collection`: creates a new collection in our STAC catalog using its human-curated jsonnet file.
- `validate-stac-collection`: validates a collection in our STAC catalog.
- `ingest-stac-collection`: ingests a collection in the PostGres database indexing our STAC catalog.
- `update-collection-from-items`: updates a collection based on the attributes of the items it contains.
- `validate-stac-item`: validates a STAC item.
- `generate-stac-item`: generates a STAC item from a `datapackage.json` file and related schema.
- `ingest-stac-item`: ingests a STAC item in the PostGres database indexing our STAC catalog.
- `delete-stac-item`: deletes a STAC item from the PostGres database indexing our STAC catalog.

### Thredds

Templates to query Thredds servers.

- `get-thredds-url`: get all the download urls for files matching a given pattern in a thredds server. Output as an artifact.
- `get-single-url`: get the download url for a single file matching a given pattern in a thredds server. Fails if more than one file is found to match. Output to stdout.

### Watchers

Templates to monitor data changes from different sources. They rely on a `watched_filenames.txt` file in the data bucket to keep track of files already ingested.

- `nasa-source-watcher`: watch the data changes happening in the NASA Earthdata platform.
- `thredds-source-watcher`: watch the data changes happening in a THREDDS server.
- `ftp-source-watcher`: watch the data changes happening in an FTP server.
- `s3-source-watcher`: watch the data changes happening in an S3 bucket.
- `ala-source-watcher`: watch the data changes happening in the Atlas of Living Australia API.
- `obis-source-watcher-byid`: watch the data changes happening in the OBIS API by dataset id.
- `obis-source-watcher`: watch the data changes happening in the OBIS API.

### Zarr

Templates to process Zarr files. Several of these templates accept multiple options as input to alter their behaviour. Check the comments in the templates' definitions for more information.

- `s3-nc-to-zarr`: convert a set of netcdf files stored on s3 to zarr.
- `gdal-to-zarr`: convert a raster file to zarr using gdal_translate.
- `gdal-merge`: merge multiple raster files into a single zarr using gdal_merge.
- `fix-coordinates`: set standard values as the attributes of coordinates of a zarr dataset.
- `append-to-txt`: concatenate two txt files. (not really related to zarr but often used to keep track of source files of zarr datasets)
- `concat-multiple-zarrs`: concatenate multiple zarr datasets into a single one.
- `rechunk-zarr`: change the chunk structure of a zarr dataset.
- `split-list`: used by partitioning zarr manager to assign partition tasks to zarr workers by splitting the full watched_list of filenames into multiple files
- `convert-attr-to-string`: convert a global or variable attribute of a zarr dataset to a string.