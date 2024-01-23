---
title: Shared Directory
keywords: ENEXA Documentation
sidebar: main_sidebar
toc: false
permalink: metadata.html
folder: mydoc
---

# Experiment Meta Data
## Image Identifiers
Using images that can be downloaded and executed is a central part of the ENEXA architecture. At the moment, the project focuses on Docker images in practice. However, the project’s meta data structure shouldn’t be bound to the Docker technology and allow the usage of other containerization technologies in the future.
The meta data of ENEXA modules has to be able to refer to Docker images that the platform should execute. Similarly, the experiment meta data has to contain the Docker images that have been executed throughout an experiment. However, the Docker image identifiers used in these two cases might be different. In practice, people use either only the image name (which is interpreted to use the latest image at run time) or an image name with a version tag. However, tags can be changed over time (the “latest” tag or major version tags are good example for changing tags), i.e., although the identifier remains the same, the image changes. Hence, for keeping track of used images, an exact identifier is needed, which can be created by combining the image name with the hash of the image. If executed, it forces Docker to use an exact version of the image. We distinguish these two types of identifiers as tagged identifiers and hashed identifiers.
While the HOBBIT project made use of literals for image identifiers, we propose to represent them as RDF resources. As IRI for these resources, the usage of a URN namepace seems to be adequate (Note that we do not work with an official name space). The URN starts with an hierarchical structure:
```
 urn:container:docker:image:
```
The term container defines the URN name space of containers, docker defines that the URN identifies an entity that is related to the Docker containerization technology and image defines that the following string identifies a Docker image. 
Our suggestion is to use one of the following two variants (<> enclose necessary information, [] encloses optional information and ()|() mark variants):
## Variant 1: One structure for both types
We define a single URN structure that covers both identifier types. The identifiers can still be distinguished by their ending.
```
urn:container:docker:image:<registry[:port]>/<path>/<image>(:<tag>)|(@<hash>)
```
Example:
```
urn:container:docker:image:docker.io/library/dicegroup/dice-embeddings:0.1.3
Variant 2: Distinct structures 
We use a prefix to define the type:
urn:container:docker:image:tag:<registry[:port]>/<path>/<image>:<tag>
urn:container:docker:image:hash:<registry[:port]>/<path>/<image>@<hash>
```
Example:
```
urn:container:docker:image:tag:docker.io/library/dicegroup/dice-embeddings:0.1.3
```

Use cases to consider for image identifiers
-	Image tag (or hash) in the module metadata, used to run the module
-	"busybox:latest" - of course modules should provide some other tag
-	Image hash stored for the provenance of runs
-	"busybox@sha256:3fbc632167424a6d997e74f52b878d7cc478225cffac6bc977eedfe51c7f4e79" - works with docker run. This is "RepoDigest" of the image, hash of the manifest. Manifest may contain many images (for example for different arch), so that's not enough
-	"sha256:3fbc632167424a6d997e74f52b878d7cc478225cffac6bc977eedfe51c7f4e79" - does NOT work with docker run, looks like the image name is required
-	"sha256:a416a98b71e224a31ee99cff8e16063554498227d2b696152a9c3e0aa65e5824" - works with docker run. This is "Id" of the image
-	"busybox@sha256:a416a98b71e224a31ee99cff8e16063554498227d2b696152a9c3e0aa65e5824" - does NOT work with docker run
-	Using the stored image hash (or other data) to reproducibly re-run the module
Notes:
-	Looks like image name is required if we actually need to "pull" the image from any registry
-	We can add both usable options ("busybox@sha256:3fb" and "sha256:a41") at the same time, since they also can be clearly distinguished and one is needed to pull the image, and another to actually reference the specific image
-	urn:container:docker:image:docker.io/library/busybox:latest
-	urn:container:docker:image:docker.io/library/busybox@sha256:3fbc632167424a6d997e74f52b878d7cc478225cffac6bc977eedfe51c7f4e79
-	urn:container:docker:image:sha256:a416a98b71e224a31ee99cff8e16063554498227d2b696152a9c3e0aa65e5824
-	we can store the content of "docker image inspect" in metadata as well

Example
In the following, we will go through a short example pipeline that is based on ENEXA. The goal of the pipeline is to generate class descriptions based on a given knowledge graph for a given set of positive and negative example entities. To this end, we first generate a knowledge graph embedding model before we run an algorithm that generates the class expressions using this model.
We will assume that there is a running Kubernetes locally. All modules will be executed locally, i.e., we won’t look into distributed setups in this example.
Note: we use turtle to represent RDF, even if the API above states to use JSON-LD. We use the following prefixes:
```
@prefix dcat:   <http://www.w3.org/ns/dcat#> .@prefix enexa:  <http://w3id.org/dice-research/enexa/ontology#> .@prefix hobbit: <http://w3id.org/hobbit/vocab#> .@prefix owl:    <http://www.w3.org/2002/07/owl#> .@prefix prov:   <http://www.w3.org/ns/prov#> .@prefix rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .@prefix rdfs:   <http://www.w3.org/2000/01/rdf-schema#> .@prefix sd:     <http://www.w3.org/ns/sparql-service-description#> .@prefix xsd:    <http://www.w3.org/2000/10/XMLSchema#> .
```
