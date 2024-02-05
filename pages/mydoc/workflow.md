---
title: Workflow
keywords: ENEXA Documentation
sidebar: mydoc_sidebar
toc: false
permalink: workflow.html
folder: mydoc
---


## Example Of One Workflow
The following example shows the general workflow of an experiment on the ENEXA platform.

1. The ENEXA service is started with the ENEXA volume mounted.
2. The Application is started with the ENEXA volume mounted.
3. The Application calls the ENEXA service to start an experiment. The service creates:
    - an identifier for the experiment,
    - a meta data file,
    - a directory in the shared volume, and
    - a triple store (optional; might be bound to an additional call).
4. The Application loads textual data into the shared volume. It may ask the ENEXA service for a directory to put the data. After that, it sends meta data information about the added data to the ENEXA service. The service adds this meta data to the internal meta data file.
5. The Application would like to extract triples from the text documents.
    - To this end, the application starts an ENEXA module for this extraction by calling the ENEXA service with an identifier for the module that it would like to start.
    - The ENEXA service starts the module (in an own container) and provides the meta data of the experiment.
    - The extraction module is configured to provide a service that can be used by the application to trigger the extraction of data from the single documents. In case the application does not provide all parameters necessary for the extraction (e.g., the output might not be specified by the application), it can try to derive the parameters from the experiment’s meta data. In this example, it uses the address of the experiment’s SPARQL endpoint as the target to which the output data will be written. In addition to the extracted triples, it adds information (e.g., provenance data) about the created graphs to the experiment’s meta data.
6. The Application asks the ENEXA service to stop the extraction module.
7. The Application would like to classify the text documents based on the extracted triples.
    - It starts an ENEXA module by calling the ENEXA service and provides a set of positive and negative examples as parameters.
    - The started ENEXA module is programmed to train a classifier based on embedding vectors of the given examples. However, from the project’s meta data, it gathers that there is no embedding model available. Hence, it starts an ENEXA module that generates knowledge graph embeddings and gives the graphs with the extracted triples from the documents as input.
    - The embedding generation module creates an embedding model, stores it in the shared volume, sends its meta data to the ENEXA service, and terminates.
    - The ENEXA class learning module loads the embeddings from the shared volume and trains a model. The model is stored in the shared volume, and its meta data are sent to the ENEXA service.
    - The application is informed that the learning ended. It starts a classification service which loads the model from the shared volume and classifies texts provided by the application.
8. The application is done and terminates the experiment. The ENEXA service terminates all modules which are still running and shuts down any services that are bound to this experiment.
