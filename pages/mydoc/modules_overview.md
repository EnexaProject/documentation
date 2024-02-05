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

### CEL Training Module Description

To initiate the CEL Training module within the ENEXA service, submit the following request to the service endpoint '/start-container'. This module, based on Class Expression Learning (CEL), is a powerful tool for automatically learning class expressions in knowledge graphs.
```
@prefix alg: <http://www.w3id.org/dice-research/ontologies/algorithm/2023/06/> .
    @prefix enexa:  <http://w3id.org/dice-research/enexa/ontology#> .
    @prefix prov:   <http://www.w3.org/ns/prov#> .
    @prefix hobbit: <http://w3id.org/hobbit/vocab#> . 
    @prefix rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
    @prefix rdfs:   <http://www.w3.org/2000/01/rdf-schema#> .
    [] rdf:type enexa:ModuleInstance ;
    enexa:experiment <[replace this with experimentIRI]> ;
    alg:instanceOf <http://w3id.org/dice-research/enexa/module/cel-train/1.0.0> ;
    <http://w3id.org/dice-research/enexa/module/cel-train/parameter/kg> <[replace with owl file iri]>;
    <http://w3id.org/dice-research/enexa/module/cel-train/parameter/kge> <[replace with embedding iri]>.
```


#### Class Expression Learning (CEL)

CEL is a machine learning method specifically tailored for learning class expressions within knowledge graphs. In the realm of knowledge graphs, class expressions serve as descriptions of the properties of entities. For instance, a class expression could define all individuals residing in a specific city or all products manufactured by a particular company.

#### Usage Guidelines

This module equips users with the capability to automatically learn complex class expressions from their knowledge graphs. By initiating the CEL Training module, users can harness machine learning techniques to derive meaningful insights and patterns from their data.

#### Getting Started

To get started with CEL Training, send the provided module instance details to the '/start-container' endpoint, ensuring to replace placeholders with the appropriate experiment and file IRIs. The module empowers users to enhance their understanding of knowledge graph entities and relationships through automated class expression learning.

### Serve after training
this module can serve a http endpoint and accept requests , for starting the service for this the bellow request should send  
```
@prefix alg: <http://www.w3id.org/dice-research/ontologies/algorithm/2023/06/> .
        @prefix enexa:  <http://w3id.org/dice-research/enexa/ontology#> .
        @prefix prov:   <http://www.w3.org/ns/prov#> .
        @prefix hobbit: <http://w3id.org/hobbit/vocab#> . 
        @prefix rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
        @prefix rdfs:   <http://www.w3.org/2000/01/rdf-schema#> .
        [] rdf:type enexa:ModuleInstance ;
        enexa:experiment <[experimentIRI]> ;
        alg:instanceOf <http://w3id.org/dice-research/enexa/module/cel-deploy/1.0.0> ;
        <http://w3id.org/dice-research/enexa/module/cel-deploy/parameter/kg> <[owl_file_iri same ]>;
        <http://w3id.org/dice-research/enexa/module/cel-deploy/parameter/kge> <[embedding_csv_iri]>;
        <http://w3id.org/dice-research/enexa/module/cel-deploy/parameter/heuristics> <[ cel_trained_file_kge_iri , it is the iri generated from last step]>.
```
after this request can send to the api  
```
http://[container name]:7860/predict
```

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

