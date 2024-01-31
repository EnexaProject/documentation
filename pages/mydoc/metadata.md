---
title: Meta Data
keywords: ENEXA Documentation
sidebar: mydoc_sidebar
toc: false
permalink: metadata.html
folder: mydoc
---

The ENEXA platform is highly dependent on using meta data. It is used to communicate with the ENEXA service, provide information for modules and keep track of files and past activities. The platform uses an RDF triple store to manage the meta data of the different experiments. In the following, we explain the ontologies that are used and how the platform generates image identifiers.

## Ontologies

The platform reuses existing ontologies but also defines new ontologies where needed. The following figure gives an overview of the Algorithm and the ENEXA ontologies.

![ENEXA ontologies](/documentation/images/ENEXA-Ontology.svg)

The purple classes origin from [PROV-O](https://www.w3.org/TR/prov-o/). The yellow and green classes belong to the Aglorithm and the ENEXA ontologies, respectively.

The prefixes used within the image and in the following are:
```
@prefix alg:    <http://www.w3id.org/dice-research/ontologies/algorithm/2023/06/> .
@prefix enexa:  <http://w3id.org/dice-research/enexa/ontology#> .
@prefix hobbit: <http://w3id.org/hobbit/vocab#> .
@prefix owl:    <http://www.w3.org/2002/07/owl#> .
@prefix prov:   <http://www.w3.org/ns/prov#> .
@prefix rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs:   <http://www.w3.org/2000/01/rdf-schema#> .
```

### Algorithm Ontology

The algorithm ontology is used by us to describe the general information about algorithms. In the sense of this ontology, an algorithm is some implementation that has the ability to take input parameters and produce results. 

#### Classes

| Name | Description |
|------|-------------|
| `alg:Algorithm` | An instance of this class is some implementation that has the ability to take input parameters and produce results. It also can typically be run several times with different parameters and different results. |
| `alg:Parameter` | This is the class of parameters that an algorithm can take. |
| `alg:Result` | This is the class of results an algorithm can produce. |
| `alg:AlgorithmDataRelation` | This is the super class of `alg:Parameter` and `alg:Result`. This super class gives the opportunity to define further classes and properties that can be used with both—parameters and results. |
| `alg:AlgorithmSetup` | An instance of this class describes a set of parameters, i.e., a setup in which an algorithm can be run. |
| `alg:AlgorithmExecution` | An instance of this class represents the execution of a particular algorithm with a particular set of parameter values and results that are produced. Since this is something that took place, it is designed as a sub class of `prov:Activity`. Algorithm executions can have sub executions that run within them. |
| `alg:Error` | This class represents errors and we plan to use it to ease the representation and retrieval of errors within the meta data graph. An error is a special case of a `prov:Entity` and is connected by `prov:wasGeneratedBy` with an `alg:AlgorithmExecution` during which it occurred. |

#### Properties

| Name | Domain | Range | Description |
|------|--------|-------|-------------|
| `alg:instanceOf` | `alg:AlgorithmExecution` | `alg:Algorithm` | This property is used to connect an algorithm execution with the algorithm that it executed. |
| `alg:parameter` | `alg:Algorithm` | `alg:Parameter` | This property connects an algorithm with the definition of one of its parameters. |
| `alg:produces` | `alg:Algorithm` | `alg:Parameter` | This property connects an algorithm with the definition of one of its results. |
| `alg:subExecution` | `alg:AlgorithmExecution` | `alg:AlgorithmExecution` | This property can be used to connect two algorithm execution instances with each other. The parent execution points to its child execution. |

### ENEXA Ontology

The algorithm ontology is used by us to describe the general information about algorithms. In the sense of this ontology, an algorithm is some implementation that has the ability to take input parameters and produce results. 

#### Classes

| Name | Description |
|------|-------------|
| `enexa:Experiment` | This class represents an experiment. |
| `enexa:Module` | This class represents a single ENEXA module. |
| `enexa:ModuleInstance` | This class represents a single execution of an ENEXA module. |

#### Properties

| Name | Domain | Range | Description |
|------|--------|-------|-------------|
| `enexa:containerId` | `enexa:ModuleInstance` | `xsd:string` | This property is used to store the container ID of an ENEXA module instance. |
| `enexa:containerName` | `enexa:ModuleInstance` | `xsd:string` | This property is used to store the container name of an ENEXA module instance. |
| `enexa:experiment` | `owl:Thing` | `alg:Parameter` | This property connects an algorithm with the definition of one of its results. |
| `enexa:location` | `prov:Entity` | `xsd:string` | This property is used to assign the location string to some artifact. The location string typically contains a location on the [shared directory](shared_directory.html). |
| `enexa:metaDataEndpoint` | `enexa:Experiment` | `owl:Thing` | This property is used to express the URL of the SPARQL endpoint that stores the meta data of an experiment. |
| `enexa:metaDataGraph` | `enexa:Experiment` | `owl:Thing` | This property is used to express the IRI of the graph in which the meta data of an experiment is stored. |
| `enexa:moduleURL` | `enexa:Module` | `owl:Thing` | This property can be used to point the ENEXA service to a module's meta data file when requesting the start of said module. |
| `enexa:sharedDirectory` | `enexa:Experiment` | `xsd:string` | This property is used to express the location of the experiment's shared directory on the ENEXA [shared directory](shared_directory.html). |

### Image Identifiers

Using images that can be downloaded and executed is a central part of the ENEXA architecture. At the moment, the project focuses on Docker images. However, the project’s meta data is designed to not be bound to the Docker technology and allows the usage of other containerization technologies in the future. The meta data of ENEXA modules has to be able to refer to Docker images that the platform should execute. 

Similarly, the experiment meta data has to contain the Docker images that have been executed throughout an experiment. However, the Docker image identifiers used in these two cases might be different. In practice, people use either only the image name (which is interpreted to use the latest image at run time) or an image name with a version tag. However, tags can be changed over time (the “latest” tag or major version tags are good example for changing tags), i.e., although the identifier remains the same, the image changes. Hence, for keeping track of used images, an exact identifier is needed, which can be created by combining the image name with the hash of the image. If executed, it forces the underlying container system (e.g., Docker) to use an exact version of the image. We distinguish these two types of identifiers as tagged identifiers and hashed identifiers.

While other projects (e.g., HOBBIT) made use of literals for image identifiers, we propose to represent them as RDF resources. As IRI for these resources, the usage of a URN namepace seems to be adequate (Note that we do not work with an official name space). The URN starts with an hierarchical structure:
```
 urn:container:docker:image:
```
The term container defines the URN name space of containers, docker defines that the URN identifies an entity that is related to the Docker containerization technology and image defines that the following string identifies a Docker image. 

Our suggestion is to use the following structure for Docker image identifiers (<> enclose necessary information, [] encloses optional information and ()|() mark variants):
```
urn:container:docker:image:(<registry[:port]>/<path>/<image>(:<tag>)|(@<hash>))|(@<hash>)
```

The first variant is the image name with a tag. For example:
```
urn:container:docker:image:docker.io/library/dicegroup/dice-embeddings:0.1.3
```

The first variant is the image name with a tag. For example:
```
urn:container:docker:image:docker.io/library/busybox:latest
```

the second variant is the image name with a hash that identifies the version of the image. for example:
```
urn:container:docker:image:docker.io/library/busybox@sha256:3fbc632167424a6d997e74f52b878d7cc478225cffac6bc977eedfe51c7f4e79
```
This image identifier can be used to download the image (e.g., using `docker pull`). It is called `RepoDigest` in the image and represents the hash of the image's manifest. However, a manifest may contain many images for different platforms and this hash alone doesn't seem to be enough to run a new instance of the image (i.e., it doesn't work with `docker run`). Hence, storing only this identifier is not enough. In addition, ENEXA stores another hash, which is represented as third variant of the general IRI defined above:
``` 
urn:container:docker:image:sha256:a416a98b71e224a31ee99cff8e16063554498227d2b696152a9c3e0aa65e5824
```
This hash cannot be used to pull the image, but it is a unique identifier for the image that has been executed.


