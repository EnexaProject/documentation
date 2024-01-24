---
title: Service Description
keywords: ENEXA Documentation
sidebar: mydoc_sidebar
toc: false
permalink: service_des.html
folder: mydoc
---

# Service Start

- The service needs to be started with a shared volume. The service has to be aware of the directory so that it can add it to
  - the meta data of experiments, and
  - newly created containers

- The service needs to be located in a virtual network and it needs to be aware of the network’s name so that it can add newly created containers to the same network.

- The service needs to be able to interact with the containerization software (e.g., Kubernetes)

# REST API

- RDF serialization:
  - By default, JSON-LD is used
  - The service may take the Accept header in the request into account to return results (e.g., RDF data) using other serializations

  
| Name | Start Experiment |
|----------|----------|
| URL | /start-experiment |
| Description | The service sets up the environment for a new experiment |
| Type |-	HTTP POST -	synchronous  |
| Parameters |  |
| Steps |1.	Generate experiment IRI and create its meta data <br> 2.	Create shared directory <br> 3.	Start default containers <br> 4.	Update experiment meta data with data from steps 2 and 3  |
| Response type| JSON-LD |
| Response content |-	Experiment IRI -	Meta data SPARQL endpoint URL -	Shared directory path  |
| Errors | ·	HTTP 500:o	The service was not able to perform one of the necessary steps |


| Name                | Meta data endpoint                                    |
|---------------------|-------------------------------------------------------|
| URL                 | /meta                                                 |
| Description         | This method returns the URL of a SPARQL endpoint and the IRI of the graph within this endpoint that contains the meta data for the experiment with the given IRI. |
| Type                | - HTTP GET                                            |
|                     | - synchronous                                         |
| Parameters          | - experiment = Experiment IRI                          |
| Steps               | 1. Look up the SPARQL endpoint URL and the graph IRI  |
| Response type       | JSON-LD                                               |
| Response content    | - Meta data SPARQL endpoint URL                       |
|                     | - Meta data graph IRI (same response schema as /start-experiment) |
| Errors              | - HTTP 400:                                           |
|                     |   - Experiment IRI is not known / not available.      |
|                     | - HTTP 500:                                           |
|                     |   - There is no such SPARQL endpoint available.       |


| Name                        | Start a module or container                                    |
|-----------------------------|---------------------------------------------------------------|
| URL                         | /start-container                                              |
| Description                 | This method starts a container with the given image name as part of the experiment with the given experiment IRI. |
| Type                        | - HTTP POST                                                    |
|                             | - asynchronous (the method does not ensure that the service or module is fully running) |
| Parameters                  | - experiment = Experiment IRI                                   |
|                             | - module-iri = the module’s IRI (can be an extension of the module’s URL below) |
|                             | - module-url = location of the module’s meta data (optional)   |
|                             | - parameters = key-value pairs of the module’s parameters (predicate-object pairs, optional) |
| Parameter type              | multipart/form-data, RDF (e.g., JSON-LD)                        |
| Steps                       | 1. Derive meta data for the module that should be started       |
|                             |   a. The module IRI is looked up in a local repository (e.g., in a set of files) or, |
|                             |   b. The module URL (if provided) is accessed via HTTP to load the module with the given IRI; |
|                             |   c. If the first two attempts are not possible or do not give any result, the module IRI is dereferenced to download the meta data. If the data contains more than one module, the “latest” (i.e., the one with the latest publication date) is used. |
|                             | 2. Create a resource for the new module instance in the meta data graph. Add the parameters and their values. |
|                             | 3. Start the image                                            |
|                             |   a. with the ENEXA environmental variables                  |
|                             |   b. as part of the local network of the ENEXA service.       |
|                             | 4. Add start time (or error code in case it couldn’t be started) to the experiment’s meta data. |
|                             | 5. Return the meta data of the newly created container (including its DNS name) |
| Response type               | JSON-LD                                                       |
| Response content            | - Meta data of the newly created container                     |
| Errors                      | - HTTP 400:                                                   |
|                             |   o Experiment IRI is not known / not available.              |
|                             |   o The image does not exist or cannot be found.               |
|                             | - HTTP 500:                                                   |
|                             |   o An error occurs while communicating with the Kubernetes service. |


| Name                        | Adds a file to an experiment                                  |
|-----------------------------|---------------------------------------------------------------|
| URL                         | /add-resource                                                 |
| Description                 | This method adds the given resource to the experiment’s meta data. Remote resources are downloaded to the provided target directory. |
| Type                        | - HTTP POST                                                    |
|                             | - synchronous                                                  |
| Parameters                  | - experiment = Experiment IRI                                   |
|                             | - resource-url = The URL of the resource (optional)            |
|                             | - target-dir = Target directory in the experiment’s shared directory |
| Parameter type              | multipart/form-data, RDF (e.g., JSON-LD)                        |
| Steps                       | 1. If the resource has a known protocol that does not start with the file protocol, the service should try to fetch the data. |
|                             | 2. Generate file name                                          |
|                             | 3. Add file to the shared directory                             |
|                             | 4. Add and return the file’s meta data                          |
| Response type               | RDF;                                                          |
|                             | Additionally, plain resource IRI in "Content-Location" HTTP header |
| Response content            | - Meta data of the newly added file                             |
| Errors                      | - HTTP 400:                                                   |
|                             |   o Experiment IRI is not known / not available.              |
|                             |   o The resource URL does not exist or cannot be downloaded.  |
|                             | - HTTP 500:                                                   |
|                             |   o An error occurs while adding the resource.                |


| Name                        | Get the status of a container                                  |
|-----------------------------|---------------------------------------------------------------|
| URL                         | /container-status                                            |
| Description                 | This method returns status information for the given container that is gathered from the Kubernetes service. |
| Type                        | - HTTP GET                                                    |
|                             | - synchronous                                                  |
| Parameters                  | - experiment = Experiment IRI                                   |
|                             | - container = Container IRI (or DNS name)                      |
| Steps                       | 1. Get the container status from the Kubernetes service and transform it into RDF |
| Response type               | Some RDF serialization                                        |
| Response content            | - The status of the container expressed as RDF. This could also express that the container does not exist. |
| Errors                      | - HTTP 400:                                                   |
|                             |   o Experiment IRI is not known / not available.              |
|                             | - HTTP 500:                                                   |
|                             |   o An error occurs while communicating with the Kubernetes service. |


| Name                        | Finish a container of the experiment                            |
|-----------------------------|---------------------------------------------------------------|
| URL                         | /finish-container                                             |
| Description                 | This method terminates a container that is part of the experiment. |
| Type                        | - HTTP POST                                                   |
|                             | - synchronous                                                  |
| Parameters                  | - experiment = Experiment IRI                                   |
|                             | - container IRI                                               |
| Parameter type              | multipart/form-data, RDF (e.g., JSON-LD)                        |
| Steps                       | 1. Iterate over the experiment’s containers and search for the container with the given IRI |
|                             | 2. Stop the container                                          |
|                             | 3. Update the experiment meta data (e.g., the file location of a SPARQL endpoint’s content) |
| Response type               | —                                                             |
| Response content            | HTTP 200                                                       |
| Errors                      | - HTTP 400:                                                   |
|                             |   o Experiment IRI is not known / not available.              |
|                             |   o Container IRI is not known / not available / does not belong to the given Experiment IRI |
|                             | - HTTP 500:                                                   |
|                             |   o An error occurs while communicating with the Kubernetes service. |


| Name                        | Finish an experiment                                           |
|-----------------------------|---------------------------------------------------------------|
| URL                         | /finish-experiment                                           |
| Description                 | This method finishes the experiment with the given IRI by stopping all its remaining containers. |
| Type                        | - HTTP POST                                                   |
|                             | - synchronous                                                  |
| Parameters                  | - experiment = Experiment IRI                                   |
| Parameter type              | multipart/form-data, RDF (e.g., JSON-LD)                        |
| Steps                       | 1. Iterate over the experiment’s containers and stop them      |
|                             | 2. Update the experiment meta data (e.g., the file location of a SPARQL endpoint’s content) |
| Response type               | —                                                             |
| Response content            | HTTP 200                                                       |
| Errors                      | - HTTP 400:                                                   |
|                             |   o Experiment IRI is not known / not available.              |
|                             | - HTTP 500:                                                   |
|                             |   o An error occurs while communicating with the Kubernetes service. |

