---
title: Platform Overview
keywords: ENEXA Documentation
sidebar: mydoc_sidebar
toc: false
permalink: overview.html
folder: mydoc
---

The main goals of the ENEXA platform is to ease the usage of the technical outcomes of the [ENEXA project](https://enexa.eu/) by coupling the technical parts through data sharing and [containerization](https://www.docker.com/what-docker). The ENEXA platform is based on the general idea that a user application comprises one or more experiments. The platform has the goal to support the execution of these experiments by providing reusable modules and a framework that supports the easy usage of these modules. The following figure shows an overview over the (simplified) architecture of the platform.

![component diagram](/images/Enexa-Architecture-Sep-23.svg)

The different colors in the figure show the four different parts of the platform. The blue layer on the bottom shows the data layer, that is used to share data and services. The green layer is the exeuction layer that contains the modules that are executed. The orange application layer on top contains the application started by the user. The purple rectangle on the side depicts the ENEXA service that offers functionalities needed by the application, e.g., to start and stop ENEXA modules. A special role plays the meta data store. It is located in the data layer and all modules and applications have access to it. However, the ENEXA service makes use of it to record the progress of an experiment and to provide necessary meta data to the different platform components.

## Application Layer

The Application Layer contains the main user application. This application makes use of the ENEXA service to create an experiment. This experiment represents the current application run. The ENEXA service will prepare the necessary environment for this particular experiment. The other two layers can be used by the application either directly (e.g., by opening files on the data layer) or through specific methods of the ENEXA service (e.g., by starting a module on the execution layer). As the main executable, the application in the application layer defines the start and end of an experiment. It is also typically the only component in the architecture that communicates directly with users, e.g., through a graphical user interface.

The ENEXA platform supports different types of applications. These types may range from pipeline applications, which process large amounts of data, to server applications, which provide a service to users. In a similar way, the application can use other libraries in parallel, i.e., not all functionalities have to be implemented as a module on the execution layer of the ENEXA platform. The ENEXA platform increases the amount of possibilities for application developers without limiting them.

## Execution Layer

The Execution Layer comprises modules that are provided by ENEXA and executed by the application using the ENEXA service. These modules offer different functionalities, e.g., extraction of structured data from unstructured data or the generation of KG embeddings (See the [list of available modules](modules_overview.html) for an overview).
The modules get their experiment-related data and meta data from the data layer (from the shared volume or from a service). Similarly, these modules should store their results on the data layer to make them available to other modules and the user application. 

It is easily possible to add new modules to the ENEXA service (See the [module development tutorial](/module_dev.html) for more details). A module can comprise a program that is executed once or a service that runs and is used by the application or other modules.

## Data Layer

The data layer is used for the synchronization of processes and ensures that all parts of the platform
work on the same data. It comprises two parts: a shared volume and a set of services.
7.4.1 Shared Volume
The shared volume contains the data of an experiment. This ranges from raw data over RDF or Web
Ontology Language (OWL) files to trained embedding or machine learning models (DL-4, DL-5). This
volume is shared with the application in the application layer and the modules in the execution layer.
Depending on the environment in which an experiment runs, the shared volume might be based on
additional services outside of the ENEXA platform. For example, in case of a distributed environment,
a solution for volumes across the machines in this environment has to be provided and configured in
addition to the ENEXA platform (e.g., Network File System (NFS); DL-13).
....................................................................................................
Page 56D2.1 - v. 1.0
....................................................................................................
7.4.2 Services
The data layer may contain several services. A service in the data layer is expected to provide data
that can be useful for the application as well as multiple modules. Examples of such a service are triple
stores providing SPARQL endpoints or an index like Elasticsearch (DL-5, EL-15). These services are
started and stopped by the ENEXA platform. To this end, services have to be provided as container
that can be executed on Kubernetes.

 components are the platform components that offer its main functionality. The orange components belong to a benchmark and are only instantiated if the benchmark is running. The grey component is the system that is benchmarked by the orange benchmark. Note that even if the overview picture shows the system as a single component, it might comprise multiple distributed components.
 
 
 
 
Each experiment is created and controller by an application.

The ENEXA platform supports the sharing of data and services between processes that are started
within the same experiment. It comprises four parts that are depicted in the Architecture.
 
 
 
 

Internally, the platform is written in Java and its communication is based on [RabbitMQ](http://www.rabbitmq.com/). However, the benchmarks and benchmarked systems can be written in any language as long as a) they are able to communicate with the platform and b) they are available as Docker containers.

## Workflow of a benchmark

It can be seen that the benchmarking of a system has three different phases. First, the system and the benchmark are created and initialized. Second, the benchmark starts to benchmark the system and stores its responses in the evaluation storage. Third, the system as well as other parts are terminated and the data in the evaluation storage is used to evaluate the system responses.

[Experiment workflow](/experiment_workflow.html) explains that in more detail.
