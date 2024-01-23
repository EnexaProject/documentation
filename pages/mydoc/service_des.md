---
title: Service Description
keywords: ENEXA Documentation
sidebar: main_sidebar
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

