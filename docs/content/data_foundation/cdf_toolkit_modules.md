### Environment configuration files

Let's start by opening the `ice-cream-dataops` repository in Visual Studio Code. Expand the nested `ice-cream-dataops`, to find the config file:

  - `config.test.yaml`

This file holds the environment variables that you will reference in the modules. You need to change the name of the `project` to your assigned CDF project (`cdf-bootcamp-<n>-<env>`).

The `selected` parameter is to indicate which modules I would like the `Cognite Toolkit` to build and deploy. To read more about this, click [here](https://docs.cognite.com/cdf/deploy/cdf_toolkit/guides/usage#step-2-configure-modules){target=_blank}.

---

### Configure Test Environment Configuration File

As a best practice, we will add the Object IDs belonging to the Microsoft Entra ID in the `config.test.yaml`. This is in case we need to change an Object ID, we can change it in one place and make sure this will be reflected across all the module configs. We can add the Object IDs under `variables:`. The variables for the groups have a suffix of `_source_id` Look at the example below.

```yaml title="config.env.yaml"
{% include "../code/data_foundation/config.env.yaml" %}
```

**Open** your `config.test.yaml` and add the Object IDs and Client IDs that we have created today. Make sure to update the parameters for `environment` at the top of the file as well.

The CDF Toolkit will walk you through setting up your configuration modules. Make sure to read the output thoroughly because there are instructions for you to follow. Before we run the first `Cognite Toolkit` command, we need to setup a `.env` that will hold the secrets to the Service Principals.

1. Create a `.env` file at the top of the repo `~/workspace/ice-cream-dataops/`.
2. Copy this into the file:

```yaml title=".env"
{% include "../code/data_foundation/env.yaml" %}
```

!!! warning
    Please be aware of this question while running the next step. If the group doesn't exist in CDF then you will need to select `Yes`.

    ```shell
    ? No cognite_toolkit_service_principal exists. Do you want to create a new group for running the toolkit with the capabilities?
    ```

    You need to pay close attention to this question, as this step will create the CDF Group `cognite_toolkit_service_principal` which has all the capabilities the `Cognite Toolkit` needs.

    There might be a warning about the Toolkit unable to check function service status. This can be safely ignored.

In a terminal, run this command to set test authentication and set up the Cognite Toolkit group in CDF:

=== "poetry"
    ```shell
    poetry run cdf-tk auth verify
    ```

=== "pip"
    ```shell
    cdf-tk auth verify
    ```

!!! tip
    You might need to change directory into the toolkit's configuration directory
    `cd ice-cream-dataops/`

---

### Configure data_foundation module

#### Modify YAML file for Groups

`admin.readonly.group.yaml` and `admin.readwrite.group.yaml` are groups pre-configured. These specific groups are meant to provide all the necessary capabilities for the toolkit.

Example of a Group

```
name: threed_engineer
sourceId: '{{source_id}}'
metadata:
  origin: cognite-toolkit
capabilities:
  - projectsAcl:
      actions: [LIST, READ]
      scope:
        all: {}
  - threedAcl:
      actions:
        - READ
      scope:
        datasetScope:
          ids: ['my_dataset']
```

---

**Define data_developer Group**

The `data_developer` group is meant for developers who will start building use cases and data pipelines. You can find [here](https://docs.cognite.com/cdf/access/guides/capabilities/#create-and-assign-security-categories){target=_blank} the list of capabilities available in Cognite Data Fusion.

1. In the folder `modules/data_foundation/auth/`, create a new YAML file with the name of the group and the `.yaml` extension and open it:

    ```
    data_developer.yaml
    ```

2. Add a property called `name` with the value `data_developer`

    ```
    name: data_developer
    ```

3. Add `sourceId`, and reference the Object ID stored in `ice-cream-dataops/config.test.yaml`

    ```
    sourceId: {{ data_developer_source_id }}
    ```

    ??? tip "Why the double brackets?"
        The double brackets let the Toolkit know that you want to use variable expansion on this property. It will replace it with the value in your config.test.yaml
        `sourceId: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

4. Add all the capabilities from `admin.readwrite.group.yaml` except the `groupsAcl` capabilities.
5. Add these extra capabilities that are required for `Diagram Parsing`

    ```
      - eventsAcl:
          actions:
            - READ
            - WRITE
          scope:
            all: {}
      - annotationsAcl:
          actions:
            - READ
            - WRITE
          scope:
            all: {}
    ```

---

**Define icapi_extractors Group**

To create the `icapi_extractors` group you will need to:

1. Create a folder `auth` in `ice_cream_api`

    ```ice-cream-dataops/modules/bootcamp/ice_cream_api/auth```

2. Create a new YAML file `icapi_extractors.yaml`
3. Add its `name` and the `sourceId` variable from `ice-cream-dataops/config.test.yaml`
4. Add these capabilities and note how they are scoped to the Ice Cream API's source data set:

    ```
      - assetsAcl:
          actions: [READ, WRITE]
          scope:
            datasetScope:
              ids: [{{ icapi_ds_external_id }}]
      - datasetsAcl:
          actions: [READ]
          scope:
            idScope:
              ids: [{{ icapi_ds_external_id }}]
      - extractionPipelinesAcl:
          actions: [READ]
          scope:
            datasetScope:
              ids: [{{ icapi_ds_external_id }}]
      - extractionRunsAcl:
          actions: [READ, WRITE]
          scope:
            datasetScope:
              ids: [{{ icapi_ds_external_id }}]
      - filesAcl:
          actions: [READ, WRITE]
          scope:
            datasetScope:
              ids: [{{ icapi_ds_external_id }}]
      - functionsAcl:
          actions: [READ, WRITE]
          scope:
            all: {}
      - groupsAcl:
          actions: [LIST, READ]
          scope:
            currentuserscope: {}
      - projectsAcl:
          actions: [LIST, READ]
          scope:
            all: {}
      - rawAcl:
          actions: [LIST, READ, WRITE]
          scope:
            tableScope:
              dbsToTables:
                ice_cream_api: [icapi_assets_extractor]
                State Store: [icapi_datapoints_extractor]
      - sessionsAcl:
          actions: [LIST, CREATE, DELETE]
          scope:
            all: {}
      - timeSeriesAcl:
          actions: [READ, WRITE]
          scope:
            datasetScope:
              ids: [{{ icapi_ds_external_id }}]
      - transformationsAcl:
          actions: [READ, WRITE]
          scope:
            datasetScope:
              ids: [{{ icapi_ds_external_id }}]
    ```

---

**Define data_pipeline_oee Group**

To create the `data_pipeline_oee` group you will need to:

1. Create a folder `auth` in `use_cases/oee`

    ```ice-cream-dataops/modules/bootcamp/use_cases/oee/auth```

2. Create a new YAML file `data_pipeline_oee.yaml`
3. Add its `name` and the `sourceId` variable from `ice-cream-dataops/config.test.yaml`
4. Add these capabilities and note how they are scoped to read from the Ice Cream API's source data set, and write to OEE's data set:

    ```
      - assetsAcl:
          actions: [READ]
          scope:
            datasetScope:
              ids: [{{ icapi_ds_external_id }}]
      - datasetsAcl:
          actions: [READ]
          scope:
            idScope:
              ids: [{{ uc_oee_ds_external_id }}, {{ icapi_ds_external_id }}]
      - groupsAcl:
          actions: [LIST, READ]
          scope:
            currentuserscope: {}
      - projectsAcl:
          actions: [LIST, READ]
          scope:
            all: {}
      - sessionsAcl:
          actions: [LIST, CREATE, DELETE]
          scope:
            all: {}
      - timeSeriesAcl:
          actions: [READ]
          scope:
            datasetScope:
              ids: [{{ icapi_ds_external_id }}]
      - timeSeriesAcl:
          actions: [READ, WRITE]
          scope:
            datasetScope:
              ids: [{{ uc_oee_ds_external_id }}]
    ```

---

#### Modify YAML file for RAW databases

We need a table to store the RAW data from the assets, and a table for the extractor to use to keep track of the datapoints it has pulled.

1. Create a folder `raw` in `modules/ice_cream_api/`

    ```
    ice-cream-dataops/modules/bootcamp/ice_cream_api/raw
    ```

2. Create a new YAML file with the name `ice_cream_api.yaml`
3. Add these 2 lines to the file:

    ```
    - dbName: ice_cream_api
      tableName: icapi_assets_extractor
    ```

4. Using the previous steps as a hint, create a RAW (Staging) database and table configuration for the `State Store` database and `icapi_datapoints_extractor` table.

    ??? hint
        We named the file the same as the database!

---

#### Modify YAML file for data sets

Similar to what we have done for RAW databases and groups, we will need to create a folder for the new resource type, in this case `data_sets`.

1. Create a `data_sets` folder under the `ice_cream_api` directory.
2. In the newly created directory, create a file named `data_sets.yaml`.
    1. For this module, we have one data set named `Ice Cream Factory Source Data`
    2. The `externalId` is used in multiple configurations so it is defined in `config.test.yaml` for reusability.
       Use the variable expansion notation that you learned when defining groups.
    3. `name` and `externalId` are required, a `description` is optional
3. Create a `data_sets` folder and `.yaml` under the `use_cases/oee` directory
    1. Create a data set named `Overall Equipment Effectiveness Use Case`
    2. Same as before, use variable expansion for the `externalId` property.

At the end of these steps, you should have a directory looking like this:

  ```
  📦ice-cream-dataops
    ┗ 📂ice-cream-dataops
      ┗ 📂modules
        ┗ 📂bootcamp
          ┣ 📂data_foundation
          ┣ 📂ice_cream_api
          ┃ ┣ 📂auth  <------------- NEW
          ┃ ┣ 📂data_sets  <-------- NEW
          ┃ ┣ 📂extraction_pipelines
          ┃ ┣ 📂functions
          ┃ ┗ 📂raw <--------------- NEW
          ┗ 📂use_cases
            ┗ 📂oee
              ┣ 📂auth <------------ NEW
              ┣ 📂data_sets <------- NEW
              ┗ 📂functions
  ```

---

### Run locally

To deploy the groups, RAW databases and data sets that we have configured, we need to tell the `cdf-tk` what needs to be deployed. For this, we will open `config.test.yaml` and add the following modules:

```
  selected:
  - modules/bootcamp/data_foundation
  - modules/bootcamp/ice_cream_api
  - modules/bootcamp/use_cases/oee
```

To build and deploy, run the following commands

=== "poetry"
    ```shell
    poetry run cdf-tk build
    ```
=== "pip"
    ```
    cdf-tk build
    ```

If the result has been succesful, run to preview

=== "poetry"
    ```shell
    poetry run cdf-tk deploy --dry-run
    ```
=== "pip"
    ```
    cdf-tk deploy --dry-run
    ```


If you get an output like the image above, means we have configured correctly the modules and we are ready to deploy.

=== "poetry"
    ```shell
    poetry run cdf-tk deploy
    ```
=== "pip"
    ```
    cdf-tk deploy
    ```

### Validate in Fusion UI

To have the necessary capabilities to vizualise Data Sets, Groups and the staging area (RAW), you need to add youself as a member of the `bootcamp-<n>-test-data-developer` group in Microsoft Entra ID. 

Login to your CDF project in [Fusion](https://cog-enablement-bootcamp.fusion.cognite.com/){target=_blank}  (organization name = `cog-enablement-bootcamp`)
and check that the following is in place:

- Switch to the `Admin` workspace. In `Access management`, all of your groups should be there along with 2 groups that we use for bootcamp administration (`oidc-admin-group` and `service_account_admin`). Scroll all the way to the bottom and verify that the Source ID matches the Object ID of the group in Microsoft Entra ID.
