The [Cognite Data Fusion (CDF) Toolkit](https://docs.cognite.com/cdf/deploy/cdf_toolkit/){target=_blank} is a command-line tool for configuring, deploying, and administering CDF projects.

The CDF Toolkit comes with pre-configured modules that you can use to quickly set up a CDF project with demo data or modify it to use your organization's data. You can also create custom modules to match your detailed needs.

For each CDF project, customize the setup with project specific configuration files that define the environment variables and list the modules to deploy.

A module is a bundle of logically connected resource configuration files, each of which sets up one CDF resource type. For example, a module can contain resource configuration files to set up the necessary extractors, extraction pipelines, data sets, and transformation resources for a complete **data pipeline.**

To learn more about customized modules, click [here](https://docs.cognite.com/cdf/deploy/cdf_toolkit/guides/modules/custom){target=_blank}.

![CDF Toolkit Templates](https://apps-cdn.cogniteapp.com/@cognite/docs-portal-images/1.0.0/images/cdf/cdf_deploy/cdf_toolkit/templates.png)

CDF Toolkit enables a good practice of setting up a configuration pipeline using CI/CD. Some guiding principles for keeping configuration as a code:
- Traceability through git commits and history
- [Four eye principal](https://www.unido.org/overview/member-states/change-management/faq/what-four-eyes-principle){target=_blank} through GitHub branch protection
- Repeatability through a configuration file that can be applied across environments

Let's go through some basic concepts:

- **Groups** control access to CDF, including access to Datasets and RAW databases; CDF groups are mapped to Microsoft Entra ID Groups
- **Datasets** group and track data by its source
  See [official Cognite's documentation](https://docs.cognite.com/cdf/data_governance/concepts/datasets/){target=_blank}
  for more information on Datasets
- **RAW database** is a staging area for data ingested into CDF before being contextualized
  See [official Cognite's documentation](https://docs.cognite.com/cdf/integration/guides/extraction/raw_explorer/){target=_blank}
  for more information on Cognite RAW

---

### Install CDF Toolkit

Make sure you met the [prerequisites](https://docs.cognite.com/cdf/deploy/cdf_toolkit/guides/setup#prerequisites){target=_blank} before you continue with the installation the CDF toolkit.

1. Install python 3.11

    ```shell
    pyenv install 3.11
    ```

2. Set 3.11 as your global python version

    ```shell
    pyenv global 3.11
    ```

3. Activate your python environment

    === "poetry"
        Follow the on-screen instructions on how to initialize poetry for a new python project:

        ```shell
        poetry init
        ```

        Now tell **`poetry`** to use the version of python that we installed with **`pyenv`**:
        ```
        poetry env use python
        poetry install --sync
        ```
    === "pip"
        ```shell
        virtualenv -p path/to/python venv
        source venv/bin/activate
        ```

4. Install the CDF Toolkit in your environment, run:

    === "poetry"
        ```shell
        poetry add cognite-toolkit
        ```
    === "pip"
        ```shell
        pip install cognite-toolkit
        ```

5. Initialize the configuration files and modules:

    === "poetry"
        ```shell
        poetry run cdf-tk modules init
        ```
    === "pip"
        ```shell
        cdf-tk modules init
        ```

When prompted for a directory name, just press `Enter`. This directory is then populated with a set of configuration files and template modules to configure for your CDF projects. The folder structure should look like this:

  ```
  📦ice-cream-dataops
  ┗ 📂ice-cream-dataops
    ┗ 📂modules
      ┗ 📂bootcamp
        ┣ 📂data_foundation
        ┃ ┣ 📂auth
        ┣ 📂ice_cream_api
        ┃ ┣ 📂extraction_pipelines
        ┃ ┣ 📂functions
        ┃ ┃ ┣ 📂icapi_assets_extractor
        ┃ ┃ ┣ 📂icapi_datapoints_extractor
        ┃ ┃ ┗ 📂icapi_timeseries_extractor
        ┗ 📂use_cases
          ┗ 📂oee
            ┗ 📂functions
              ┗ 📂oee_timeseries
  ```

---

### What are YAML files and their function

YAML is a human-readable data serialization language often used for writing configuration files. It is a popular programming language because it is designed to be easy to read and understand. YAML files use a .yml or .yaml extension and follow specific syntax rules. There are no usual format symbols, such as braces, square brackets, closing tags, or quotation marks. YAML files are simpler to read as they use Python-style indentation to determine the structure and indicate nesting.

The YAML resource configuration files are core to the CDF Toolkit. Each file configures one of the resource types supported by the CDF Toolkit and the CDF API. This article describes how to configure the different resource types. The CDF Toolkit bundles logically connected resource configuration files in modules, and each module stores the configuration files in directories corresponding to the resource types, called resource directories.

You can find here a [list](https://docs.cognite.com/cdf/deploy/cdf_toolkit/references/resource_library){target=_blank} with examples of YAML configuration files as reference for the cdf-tk.
