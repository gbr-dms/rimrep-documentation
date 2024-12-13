# How to write a new data ingestion workflow

When writing a workflow to ingest new data, a common structure should be used to ensure consistency and compatibility with the existing templates and codes. This document outlines the attributes that should be set in the workflow and the structure that should be followed. A basic knowledge of the Argo workflow system is assumed.

## Workflow structure

The general structure of a workflow should be as follows (anything between angle brackets should be replaced with the value needed for the specific dataset):

```yaml
# <link to github dataset issue>
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: <dataset id>-
spec:
  entrypoint: <main template name>
  serviceAccountName: data-public
  arguments:
    parameters:
      - name: dataset-id
        value: "<dataset id>"
      - name: format
        value: "<parquet or zarr>"
      - name: coordinates
        value: |
          <json dictionary with keys: x,y,z,t,geometry (only those that are present in the dataset) and as values the names of the columns/variables in the dataset that correspond to those coordinates.>

      - <any other parameters that need to be referenced by the templates>

  templates:
    - name: <main template name>
        steps:
            - - <insert here any number of steps needed to convert the data>
            - - <...>
            - - <...>
            - - name: finalise
                templateRef:
                    name: tools.finalise
                    template: finalise
                arguments:
                    parameters:
                        - name: coordinates
                          value: "{{workflow.parameters.coordinates}}"
                    artifacts:
                        - name: data
                          from: "{{<reference to the converted data artifact, e.g. steps.convert.outputs.artifacts.output>}}"
    - <other templates called by the main template can be added if needed>
```

## Cron workflows

Cron workflows (running automatically at fixed times) follow the same structure, with a slightly different header and the addition of an exit handler to notify slack if the workflow fails:

```yaml
# <link to github dataset issue>
apiVersion: argoproj.io/v1alpha1
kind: CronWorkflow
metadata:
  name: <dataset id>
  namespace: argo
spec:
  schedule: <cron time specification, e.g. "15 02 * * *" to run at 2:15AM every day>
  concurrencyPolicy: "Replace"
  timezone: Australia/Sydney
  successfulJobsHistoryLimit: 2
  failedJobsHistoryLimit: 1
  workflowSpec:
    onExit: exit-handler

    <everything under `spec` in the basic workflow structure goes here, the following template is added to the `templates` section>

        - name: exit-handler
            steps:
            - - name: send-wf-status-to-slack
                when: "{{workflow.status}} != Succeeded"
                templateRef:
                    name: tools.notifications
                    template: exit-handler-slack
```

## Remote data

Some workflows will edit the data directly on s3 instead of producing a local data artifact (for example because they are updating a very large dataset).
In this case the `finalise` step should include the `remote` parameter set to `true`, and omit the `data` artifact:

```yaml
- - name: finalise
    templateRef:
        name: tools.finalise
        template: finalise
    arguments:
        parameters:
            - name: coordinates
              value: "{{workflow.parameters.coordinates}}"
            - name: remote
              value: "true"
```

## Using a WorkflowTemplate

When the same workflow can be used to ingest multiple datasets, it is recommended to build a `WorkflowTemplate` with all the steps needed, and then reference to it in a workflow for each of the datasets. Everything remains the same, except that the `templates` section is replaced by a `workflowTemplateRef` section, containing only the name of the `WorkflowTemplate` to use:

```yaml
workflowTemplateRef:
      name: <WorkflowTemplate name>
```

Note: a `WorkflowTemplate` is a completely different concept from a `Template`, the naming is unfortunate but the first is a reusable complete workflow structure, the second is akin to a function that can be called as a step in a workflow.
