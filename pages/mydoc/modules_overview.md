---
title: Modules overview
keywords: ENEXA Documentation
sidebar: mydoc_sidebar
toc: false
permalink: modules_overview.html
folder: mydoc
---
# Modules
On these pages, you'll find a variety of ENEXA modules listed below, showcasing the diverse functionalities the platform has to offer.

## Transform module

## KG fixing module

## Extraction module

## Dice embeddings module

## Dice CEL module

## TENTRIS module
### what is this module 
### Tentris Module Description

The ENEXA module, instantiated with the Tentris RDF triple store, provides a comprehensive guide on leveraging Tentris—a robust, tensor-based RDF triple store. Tentris seamlessly integrates into the ENEXA service, offering efficient and high-performance capabilities for handling RDF data.

#### Overview

Tentris is specifically designed to handle RDF data using a tensor-based approach. This module elucidates the key aspects of Tentris, emphasizing its functionalities, optimal performance, and support for SPARQL queries.


#### Key Features

1. **Speed and Efficiency:** Tentris excels in speed, ensuring swift execution of RDF queries and operations. This feature contributes to the overall efficiency of ENEXA experiments.

2. **Tensor-Based Approach:** Tentris employs a tensor-based model for storing and processing RDF triples, enhancing scalability and facilitating complex data analysis.

3. **SPARQL Support:** Users can take advantage of Tentris's comprehensive support for SPARQL queries, enabling them to seamlessly interact with RDF data and retrieve relevant information.


### how to run 
To start the ENEXA module within the service, send the following request to the
```
/start-container
``` 
endpoint. Make sure that the specified file, indicated by the instance IRI in <[this should replace with the instance IRI which contains the file]>, has been previously added to the service using the 
```
/add-resource
```
endpoint. The initiation of execution involves linking the ENEXA module to a specific experiment identified by <[this should replace with the experimentIRI]>. The relevant RDF triples are detailed below: 
```
@prefix alg: <http://www.w3id.org/dice-research/ontologies/algorithm/2023/06/> .
    @prefix enexa:  <http://w3id.org/dice-research/enexa/ontology#> .
    @prefix prov:   <http://www.w3.org/ns/prov#> .
    @prefix hobbit: <http://w3id.org/hobbit/vocab#> . 
    @prefix rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
    [] rdf:type enexa:ModuleInstance ;
    enexa:experiment <[this Should replace with the experimentIRI]> ;
    alg:instanceOf <http://w3id.org/dice-research/enexa/module/tentris/0.2.0-SNAPSHOT-1> ;
    <http://w3id.org/dice-research/enexa/module/tentris/parameter/file> <[this should replace with the instance IRI which contains the file ]>.
```

[TENTRIS Repository](https://github.com/dice-group/tentris)

