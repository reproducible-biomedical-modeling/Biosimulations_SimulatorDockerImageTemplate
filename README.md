# Template repository for a command-line program and Dockerfile for a BioSimulators-compliant simulator

This repository provides a template Python code for building a BioSimulators-compliant command-line interface for a biosimulation tool and a Dockerfile for building a BioSimulators-compliant container for the tool. More documentation about the interfaces that containerized simulators must implement is available at https://biosimulators.org/help. Several examples of BioSimulators-compliant command-line interfaces and Dockerfiles are available in the [BioSimulators GitHub organization](https://github.com/biosimulators/). A registry of compliant simulation tools is available at https://biosimulators.org. Information about submitting containerized simulators to the registry is available at https://biosimulators.org/help.

This repository is intended for developers of simulation software programs. We recommend that end-users utilize containerized simulators through the web-based graphical interfaces at https://submit.biosimulators.org and https://biosimulators.org. https://submit.biosimulators.org provides a simple web application for executing simulations and retrieving their results. https://biosimulators.org provides a more comprehensive platform for sharing and executing entire modeling studies. Instructions for using containers to execute simulations locally on your own machine are available at https://biosimulators.org/help.

## Contents
* [Building a Docker image for a simulator using this template](#building-a-docker-image-for-a-simulator-using-this-template)
* [Running containerized simulators](#running-containerized-simulators)
* [Example Docker images for simulators](#example-docker-images-for-simulators)
* [License](#license)
* [Development team](#development-team)
* [Contributing to the template](#contributing-to-the-template)
* [Acknowledgements](#acknowledgements)
* [Questions and comments](#questions-and-comments)

## Building a Docker image for a simulator using this template
1. Install the [Docker engine](https://www.docker.com/).

2. Fork this repository.

3. Create a BioSimulators-compliant command-line interface to your simulator.

   The interface should accept two keyword arguments:

   - `-i`, `--archive`: A path to a COMBINE archive which contains descriptions of one or more simulation tasks.
   - `-o`, `--out-dir`: A path to a directory where the outputs of the simulation tasks should be saved. Reports should be saved in the PyTables dialect of the Hierarchical Data Format 5 (HDF5) format, and plots should be saved in Portable Document Format (PDF) bundled into a zip archive. Reports should be saved to `{ out-dir }/reports.h5` and plots should be saved to `{ out-dir }/plots.zip`. Within each HDF5 file and zip archive, reports and plots should be saved to paths equal to the relative path of the parent SED-ML document within the parent COMBINE/OMEX archive and the id of the report/plot.

     The rows of report tables should correspond to the datasets (`sedml:dataset`) specified in the SED-ML definition of the report. The heading of each row should be the id of the corresponding dataset. Report tables of steady-state simulations should have a single column of the steady-state predictions of each dataset. Report tables of one step simulations should have two columns that represent the predicted start and end states of each dataset. Report tables of time course simulations should have multiple columns that represent the predicted time course of each dataset. Report tables of non-spatial simulations should not have additional dimensions. Report tables of spatial simulations should have additional dimensions that represent the spatial axes of the simulation.

   In addition, we recommend providing handlers for reporting help and version information about the command-line interface to your simulator:

   - `-h`, `--help`: This argument should instruct the command-line program to print help information about itself.
   - `-v`, `--version`: This argument should instruct the command-line program to report version information about itself.

   This repository contains sample code for using Python and [BioSimulators utils](https://github.com/biosimulators/Biosimulators_utils) to create a BioSimulators-compliant command-line interface for a simulator. This code is located at `my_simulator/__main__.py`. To follow this example,

   1. Rename the `my_simulator` directory.
   2. Edit the name, URL of the simulator in `my_simulator/__main__.py`.
   3. Implement the `exec_sedml_docs_in_combine_archive` method in `my_simulator/core.py`. [`biosimulators_utils`](https://github.com/biosimulators/Biosimulators_utils) provides several utility methods and data structures for parsing COMBINE archives and SED-ML documents; representing archives and simulation experiments; and orchestrating the execution of all of the tasks in a simulation experiment. These utility methods make it easy for developers to handle COMBINE/OMEX-encoded archives and SED-ML-encoded simulation experiments.

   This code will produce a command-line interface similar to that below:
   ```
   usage: <my-simulator> [-h] [-d] [-q] -i ARCHIVE [-o OUT_DIR] [-v]

   BioSimulators-compliant command-line interface to the <MySimulator> simulation program <http://url.to.my.simulator>.

   optional arguments:
     -h, --help            show this help message and exit
     -d, --debug           full application debug mode
     -q, --quiet           suppress all console output
     -i ARCHIVE, --archive ARCHIVE
                           Path to OMEX file which contains one or more SED-ML-
                           encoded simulation experiments
     -o OUT_DIR, --out-dir OUT_DIR
                           Directory to save outputs
     -v, --version         show program's version number and exit
   ```

4. Optionally, package the command-line interface for easy distribution and installation.

   This repository contains sample files for packaging the sample Python-based command-line interface for distribution via [PyPI](https://pypi.python.org/) and installation via [pip](https://pip.pypa.io/en/stable/).

   - `my_simulator/_version.py`: Set the `__version__` variable to the version of your simulator.
   - `setup.py`: Edit the installation script for the command-line interface to your simulator.
   - `requirements.txt`: Edit the list of the requirements of the command-line interface to your simulator.
   - `MANIFEST.in`: Edit the list of additional files that should be distributed with the command-line interface to your simulator.
   - `setup.cfg`: This describes the wheel configuration for distributing the command-line interface to your simulator. For most command-line interfaces, this file doesn't need to be edited.

5. Create a Dockerfile for building a Docker image for the command-line interface to your simulator. [`Dockerfile`](Dockerfile) contains a template Dockerfile for a command-line interface implemented with Python.

   - Additional files that need to be copied into the image can be saved to a directory such as `assets/`.
   - In most cases, commercial licenses needed to run the image should be injected at runtime. Please contact the [BioSimulations Team](info@biosimulations.org) to discuss your needs.
   - Use Open Containers Initiative and BioContainers-style labels to capture metadata about the image. See the [Open Containers Initiative](https://github.com/opencontainers/image-spec/blob/master/annotations.md) and [BioContainers](https://biocontainers-edu.readthedocs.io/en/latest/what_is_biocontainers.html#create-a-dockerfile-recipe) documentation for more information.
     ```
     LABEL \
       org.opencontainers.image.title="BioNetGen" \
       org.opencontainers.image.version="2.5.1" \
       org.opencontainers.image.revision="2.5.1"
       org.opencontainers.image.description="Open-source software package for rule-based modeling of complex biochemical systems" \
       org.opencontainers.image.url="https://bionetgen.org/" \
       org.opencontainers.image.documentation="https://bionetgen.org/" \
       org.opencontainers.image.source="https://github.com/biosimulators/biosimulators_bionetgen" \
       org.opencontainers.image.authors="BioSimulators Team <info@biosimulators.org>" \
       org.opencontainers.image.vendor="BioSimulators Team" \
       org.opencontainers.image.licenses="MIT" \
       org.opencontainers.image.created="2020-11-11 10:48:55-05:00" \
       \
       base_image="ubuntu:18.04" \
       version="0.0.1" \
       software="BioNetGen" \
       software.version="2.5.1" \
       about.summary="Open-source software package for rule-based modeling of complex biochemical systems" \
       about.home="https://bionetgen.org/" \
       about.documentation="https://bionetgen.org/" \
       about.license_file="https://github.com/RuleWorld/bionetgen/blob/master/LICENSE" \
       about.license="SPDX:MIT" \
       about.tags="rule-based modeling,kinetic modeling,dynamical simulation,systems biology,biochemical networks,BNGL,SED-ML,COMBINE,OMEX,BioSimulators" \
       extra.identifiers.biotools="bionetgen" \
       maintainer="Jonathan Karr <karr@mssm.edu>"
     ```

6. Build the Docker image for the command-line interface to your simulator. For example, run the following command:
   ```
   docker build \
     --tag <owner>/<my_simulator>:<version> \
     --tag <owner>/<my_simulator>:latest \
     .
   ```
7. Push the Docker image to an image registry such as Docker Hub or the GitHub Container Registry. For example, run the following command:
   ```
   docker login
   docker push ghcr.io/<owner>/<repo>/<my_simulator>:latest
   ```

8. Enter metadata about your simulator into [`biosimulators.json`](biosimulators.json). This should include attributes such as those listed below. Attributes marked with `*` are optional. The schema is available in the `Schemas` >> `Simulator` section at https://api.biosimulators.org.
  - `id`: A unique id for the simulator (e.g., `tellurium`). The id must begin with a letter or underscore and include only letters, numbers, and underscores.
  - `version`\*: Version of the simulator (e.g., `1.0.0`).
  - `name`\*: Short name of the simulator.
  - `description`\*: Extended description of the simulator.
  - `image`: Docker image for the simulator
    - `url`: URL for the image (e.g., `ghcr.io/biosimulators/biosimulators_tellurium/tellurium:2.1.6`). This should include the organization which owns the image, the id of the image, and the version tag of the image.
    - `format`
      - `namespace`: `EDAM`
      - `id`: `format_3973`
      - `version`: null
      - `supportedFeatures`: []
  - `url`\*: list of URLs relevant to the simulator.
     - `type`: type (e.g., `Home page`, `Documentation`)
     - `title`: description of the URL
     - `url`: URL
  - `algorithms`: List of simulation algorithms supported by the simulator. Each algorithm should include the following information.
    - `id`: Internal id for the algorithm within the simulator (e.g., `nleq2`).
    - `name`: Optional, short name of the implementation of the algorithm in the simulator.
    - `kisaoId`: KiSAO term for the implementation of the algorithm in the simulator (e.g., `{"namespace": "KISAO", "id": "KISAO_0000057"}`).
      - `namespace`: `KISAO`
      - `id`: id of a KiSAO algorithm term (e.g., `KISAO_0000029`)
    - `modelingFrameworks`: List of modeling frameworks (e.g., flux balance analysis) supported by the implementation of the algorithm in the simulator (e.g., `[{"namespace": "SBO", "id": "SBO_0000624"}]`).
    - `parameters`: List of parameters of the implementation of the algorithm in the simulator. Each parameter should include the following information.
      - `id`: Internal id for the parameter within the algorithm (e.g., `abs_tol`).
      - `name`\*: Optional, short name of the parameter (e.g., `absolute tolerance`).
      - `kisaoId`: KiSAO term for parameter (e.g., `[{"namespace": "KISAO", "id": "KISAO_0000057"}]`).
      - `type`: Type of the parameters (`boolean`, `integer`, `float`, or `string`).
      - `value`: Default value of the parameter (e.g., `1e-6`).
      - `recommendedRange`: List of the recommended minimum and maximum values of the parameter (e.g., `["1e-3", "1e-9"]`).
    - `modelFormats`: List of model formats (e.g., CellML, SBML) supported by the implementation of the algorithm in the simulator (e.g., `[{"namespace": "EDAM", "id": "format_2585", "version": null, "supportedFeatures": []}]`).
    - `simulationFormats`: List of simulation formats (e.g., SED-ML) supported by the implementation of the algorithm in the simulator (e.g., `[{"namespace": "EDAM", "id": "format_3685", "version": null, "supportedFeatures": []}]`).
    - `archiveFormats`: List of archive formats (e.g., COMBINE) supported by the implementation of the algorithm in the simulator (e.g., `[{"namespace": "EDAM", "id": "format_3686", "version": null, "supportedFeatures": []}]`).
    - `citations`\*: List of citations for the algorithm. See `biosimulators.json` for examples.
  - `authors`\* List of the authors of the simulator
    - `firstName`
    - `middleName`
    - `lastName`
    - `identifiers`: list of identifiers (e.g., `[{"namespace": "orcid", "id": "XXXX-XXXX-XXXX-XXXX", "url": "https://orcid.org/XXXX-XXXX-XXXX-XXXX"}]`).
  - `references`\*: References for the simulator.
    - `identifiers`\*: List of identifiers (e.g., bio.tools id, BioContainers id) for the simulator (e.g., `[{"namespace": "bio.tools", "id": "bionetgen", "url": "https://bio.tools/bionetgen"}]`).
    - `citations`\*: List of citations for the simulator. See `biosimulators.json` for examples.
  - `license`: One of the licenses supported by SPDX (e.g., `{"namespace": "SPDX", "id": "MIT"}`). The list of supported licenses is available at https://spdx.org.
  - `biosimulators`\*:
    - `specificationVersion`\*: Version of BioSimulators supported by the container (e.g., `1.0.0`).
    - `imageVersion`\*: Version of the container (e.g., `1.0.0`).
    - `created`\*: Date that the image was created (e.g., `2020-10-26T12:00:00Z`).
    - `updated`\*:  Date that the image was last updated (e.g., `2020-10-26T12:00:00Z`).

  As necessary, [request additional SED-ML URNs for model formats](https://github.com/SED-ML/sed-ml/issues), [request additional COMBINE specification URLs for model formats](https://github.com/sbmlteam/libCombine/issues), and [request additional KiSAO terms for algorithm parameters](https://sourceforge.net/p/kisao/feature-requests/new/).

9. Implement tests for the command-line interface to your simulator in the `tests` directory.

   `tests/test_all.py` contains an example for testing a command-line interface implemented in Python to a simulator that supports SBML-encoded kinetic models. The `test_validator` method illustrates how to use the simulator validator. Example files needed for the tests can be saved to `tests/fixtures/`. `tests/requirements.txt` contains a list of the dependencies of these tests.

10. Replace this file (`README.md`) with `README.template.md` and fill out the template with information about your simulator.

11. Enter the name of the owner of your simulator and the year into the MIT License template at [`LICENSE.template`](LICENSE) and rename the template to `LICENSE`, or copy your license into `LICENSE`. We recommend using a permissive license such as the [MIT License](https://opensource.org/licenses/MIT).

12. Optionally, setup continuous integration for your simulation tool.

    `.github/workflows/ci.yml.template` contains a sample continuous integration workflow for GitHub Actions. The workflow executes the following tasks each time commits are pushed to your repository:
    1. Clones your repository
    2. Installs your package and its dependencies
    3. Uses [flake8](https://flake8.pycqa.org/) to lint your package.
    4. Builds the Docker image for your package and tags the image `ghcr.io/<owner>/<repo>/<simulator_id>:<simulator_version>` and `ghcr.io/<owner>/<repo>/<simulator_id>:latest`.
    5. Uses [pytest](https://docs.pytest.org/) to run the unit tests for your package and save the coverage.
    6. Uploads the coverage data to [Codecov](https://codecov.io/).
    7. Uses [Sphinx](https://www.sphinx-doc.org/) to compile the documentation for your package.

    Each time you add a tag to your repository (`git tag ...; git push --tags`), the workflow also runs the above tasks. If the above tasks succeed, the workflow executes these additional tasks:
    1. Creates a GitHub release for the tag.
    2. Pushes the compiled documentation to the repository (e.g., so it can be served by GitHub pages).
    3. Builds your package and submits it to [PyPI](https://pypi.org/).
    4. Pushes your Docker image to the [GitHub Container Registry](https://docs.github.com/en/free-pro-team@latest/packages/guides/about-github-container-registry) with the above tags. Once your image is pushed, it will be visible at `https://github.com/orgs/<org>/packages?repo_name=<repo>`.
    5. Pushes your simulator to the BioSimulators Registry by using the GitHub API to create an issue to add a new version of your simulator to the BioSimulators database. This issue will then automatically use the BioSimulators test suite to validate your simulator and add a new version of your simulator to the database if your simulator passes the test suite.

    Follow the steps below to use this workflow.
    1. Rename the file to `.github/workflow/ci.yml`
    2. Add the following secrets to the settings for your repository:
      * `CODECOV_TOKEN`: Token for submitting coverage data to Codecov. You can generate a token by creating an account, logging in, and adding this repository to CodeCov (eg., by visiting `https://codecov.io/gh/<owner>/<repo>`). If you change the default branch for your repository, tt also may be necessary to explicitly sets this in Codecov.
      * `PYPI_TOKEN`: Token for submitting your package to PyPI. You can create tokens from your account settings page (https://pypi.org/manage/account/).
      * `DOCKER_REGISTRY_USERNAME`: User name for the Docker registry (e.g., Docker Hub or GitHub Container Registry) where you want to push your Docker images.
      * `DOCKER_REGISTRY_TOKEN`: Password for the above user. For GitHub Container Registry, you can create a token from the developers settings (https://github.com/settings/tokens). The token should have scopes `repo` and `write:package`.
      * `GH_ISSUE_USERNAME`: GitHub user name to post issues to register new versions of your simulator with BioSimulators (e.g., `jonrkarr`)
      * `GH_ISSUE_TOKEN`: Token for the above GitHub user. For GitHub Container Registry, you can create a token from the developers settings (https://github.com/settings/tokens). The token should have scope `repo`.

12. Optionally, distribute the command-line interface to your simulator. For example, the following commands can be used to distribute a command-line interface implemented with Python via [PyPI](https://pypi.python.org/).
    ```
    # Convert README to RST format
    pandoc --to rst --output README.rst README.md

    # Package command-line interface
    python3 setup.py sdist
    python3 setup.py bdist_wheel

    # Install twine to upload packages to PyPI
    pip3 install twine

    # Upload packages to PyPI
    twine dist/*
    ```

## Running containerized simulators

Simulator Docker images can be run as indicated below:
```
docker run \
  --tty \
  --rm \
  --mount type=bind,source="$(pwd)"/tests/fixtures,target=/root/in,readonly \
  --mount type=bind,source="$(pwd)"/tests/results,target=/root/out \
  <registry e.g., docker.io or ghcr.io>/<organization>/<repository>/<repository> \
    -i /path/to/archive.omex \
    -o /path/to/output
```

## Example Docker images for simulators

The following are several examples of BioSimulators-compliant command-line interfaces and Docker images for biosimulation tools:
- [BioNetGen](https://bionetgen.org): [Command-line interface and Dockerfile](https://github.com/biosimulators/Biosimulators_BioNetGen), [Docker image](https://github.com/orgs/biosimulators/packages/container/package/bionetgen)
- [GillesPy2](https://github.com/StochSS/GillesPy2): [Command-line interface and Dockerfile](https://github.com/biosimulators/Biosimulators_GillesPy2), [Docker image](https://github.com/orgs/biosimulators/packages/container/package/gillespy2)
- [tellurium](http://tellurium.analogmachine.org): [Command-line interface and Dockerfile](https://github.com/biosimulators/Biosimulators_tellurium), [Docker image](https://github.com/orgs/biosimulators/packages/container/package/tellurium)

## License
This template is released under the [MIT license](LICENSE).

## Development team
This template was developed by the [Center for Reproducible Biomedical Modeling](http://reproduciblebiomodels.org) and the [Karr Lab](https://www.karrlab.org) at the Icahn School of Medicine at Mount Sinai in New York.

## Contributing to the template
We enthusiastically welcome contributions to the template! Please see the [guide to contributing](CONTRIBUTING.md) and the [developer's code of conduct](CODE_OF_CONDUCT.md).

## Acknowledgements
This work was supported by National Institutes of Health awards P41EB023912 and R35GM119771 and the Icahn Institute for Data Science and Genomic Technology.

## Questions and comments
Please contact the [BioSimulators Team](mailto:info@biosimulators.org) with any questions or comments.
