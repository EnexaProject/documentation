---
title: Environmental Variables
keywords: ENEXA Documentation
sidebar: mydoc_sidebar
toc: false
permalink: parameters_env.html
folder: mydoc
---


| Variable                        | Description                                                                                           |
|---------------------------------|-------------------------------------------------------------------------------------------------------|
| ENEXA_EXPERIMENT_IRI            | The IRI of the experiment this module is a part of.                                                   |
| ENEXA_META_DATA_ENDPOINT         | The URL of the SPARQL endpoint with the experiment’s meta data. May include credentials in the URL.  |
| ENEXA_META_DATA_GRAPH            |                                                                                                       |
| ENEXA_MODULE_INSTANCE_IRI        | The IRI of this module within the meta data graph.                                                    |
| ENEXA_SHARED_DIRECTORY           | The local (absolute) path of the shared directory within this module.                                 |
| ENEXA_WRITEABLE_DIRECTORY         | The local (absolute) path of the per-application or per-experiment writable directory within this module. This directory is typically inside ENEXA_SHARED_DIRECTORY. |
| ENEXA_MODULE_INSTANCE_DIRECTORY   | The local (absolute) path of a directory that a module can use for its own files (no temporary files!). This directory is typically inside ENEXA_WRITEABLE_DIRECTORY. |
| ENEXA_SERVICE_URL                | The URL of the ENEXA service. The following "/" is important                                          |


-ENEXA service variables

| Variable                   | Description                                                                    |
|----------------------------|---------------------------------------------------------------------------------|
| ENEXA_MODULE_DIRECTORY      | Directory containing TTL files that define (local) ENEXA modules.              |
| ENEXA_SHARED_VOLUME_NAME     | Name of the shared volume that should be mounted to all containers.             |
| ENEXA_CONTAINER_MANAGEMENT   | docker (default) or kubernetes                                                 |
