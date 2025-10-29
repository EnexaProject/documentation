---
title: Modules overview
keywords: ENEXA Documentation
sidebar: mydoc_sidebar
toc: false
permalink: modules_overview.html
folder: mydoc
---

On these pages, you'll find a variety of ENEXA modules listed below, showcasing the diverse functionalities the platform has to offer.

## Transform module

The ENEXA RDF Transformation module is a straightforward tool designed for transforming RDF data. It takes one or more RDF files as input and generates a single RDF file as output. This module is particularly useful for consolidating RDF data from multiple sources into a unified format.

### Goal

The primary objective of this module is to merge RDF datasets into a single RDF file while ensuring compatibility with Apache Jena-supported RDF serializations. Key considerations for input files include:

- **Supported RDF Serialization:** Input files must utilize an RDF serialization supported by Apache Jena.
- **Compression:** Input files can be compressed with GZIP or BZip2 for efficient data storage and transmission.
- **Metadata Graph:** Input files should include mime type information in the metadata graph. If absent, the module will infer the RDF serialization based on the file extension.

The output file:

- Contains all triples from the input RDF datasets without deduplication.
- Adheres to the specified RDF serialization, supporting streamable formats such as Turtle, N-Triples, N-Quads, and TriG.

### Parameters
All parameters use the namespace `http://w3id.org/dice-research/enexa/module/transform/parameter/`.

- **input**: Input file(s) that should be transformed. 
- **outputMediaType**: The target IANA media type that the generated file should have.

### Execution 
you can send request like bellow to '/start-container' api
```ttl
@prefix alg: <http://www.w3id.org/dice-research/ontologies/algorithm/2023/06/> .
@prefix enexa:  <http://w3id.org/dice-research/enexa/ontology#> .
@prefix prov:   <http://www.w3.org/ns/prov#> .
@prefix hobbit: <http://w3id.org/hobbit/vocab#> . 
@prefix rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .

[] rdf:type enexa:ModuleInstance ;
enexa:experiment <[Experiment IRI]> ;
alg:instanceOf <http://w3id.org/dice-research/enexa/module/transform/0.0.1> ;
<http://w3id.org/dice-research/enexa/module/transform/parameter/input> <[First input]>;
<http://w3id.org/dice-research/enexa/module/transform/parameter/input> <[Second input]>;
<http://w3id.org/dice-research/enexa/module/transform/parameter/outputMediaType> <https://www.iana.org/assignments/media-types/application/owl+xml>.

```
### Additional Information

## KG fixing module

### Goal

The ENEXA KG fixing module detects formal inconsistencies in a KG and can be configured to apply different fixing strategies to render it formally consistent.
For formally inconsistent KGs the reasoning process cannot produce any useful results, thus it is necessary to correct them, or to rely on inconsistency-tolerant reasoners, which nevertheless are typically more expensive in terms of time.
Importantly, this module allows the user to enable parallel execution so that it can process and fix web-scale KGs in a time-effective manner.
It is implemented in the Java programming language and incorporates the [OWL API](http://owlcs.github.io/owlapi/).

### Module Details
The output file includes the KG in .ttl format, free of formal inconsistency provided that the corresponding configuration parameters are enabled. 

### Parameters
All parameters use the namespace `http://w3id.org/dice-research/enexa/module/kg-fixing/parameter/`.

- **t-boxFile**: The file containing the T-Box of the graph.
- **a-boxFile**: The file containing the A-Box of the graph.

### Execution 
You can send a request like the following to '/start-container' api:
```ttl
@prefix alg: <http://www.w3id.org/dice-research/ontologies/algorithm/2023/06/> .
@prefix enexa:  <http://w3id.org/dice-research/enexa/ontology#> .
@prefix prov:   <http://www.w3.org/ns/prov#> .
@prefix hobbit: <http://w3id.org/hobbit/vocab#> . 
@prefix rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs:   <http://www.w3.org/2000/01/rdf-schema#> .

[] rdf:type enexa:ModuleInstance ;
enexa:experiment <[replace this with experimentIRI]> ;
alg:instanceOf <http://w3id.org/dice-research/enexa/module/kg-fixing/1.0.0> ;
<http://w3id.org/dice-research/enexa/module/kg-fixing/parameter/t-boxFile> <[replace with T-Box input file IRI]>;
<http://w3id.org/dice-research/enexa/module/kg-fixing/parameter/a-boxFile> <[replace with A-Box input file IRI]>.
```
### Additional Information
More details regarding the usage and available configurations can be found in the README of [this repository](https://github.com/xarakas/kg-fixing/).

## Extraction module

### Goal
The ENEXA extraction module aim is to extract a knowledge graph from individual text. This includes both named entity recognition, relation extraction and entity linking to Wikidata. The overall systems provides flexible types and relations allowing users to define these on an as needed basis. 

### Parameters

- **prompt_template:** the prompt template to use for extraction
- **target_entity_types:** The target entity types to extract 
- **target_relations:** The target relations to extract
- **LLM:** The Hugging Face model to use
- **max_tokens:** The maximum number of tokens to use

### Execution
```ttl
@prefix alg:    <http://www.w3id.org/dice-research/ontologies/algorithm/2023/06/> .
@prefix enexa:  <http://w3id.org/dice-research/enexa/ontology#> .
@prefix hobbit: <http://w3id.org/hobbit/vocab#> .
@prefix owl:    <http://www.w3.org/2002/07/owl#> .
@prefix rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs:   <http://www.w3.org/2000/01/rdf-schema#> .

[] rdf:type enexa:ModuleInstance ;
enexa:experiment <[replace this with experimentIRI]> ;
alg:instanceOf <http://w3id.org/dice-research/enexa/module/enexa-extraction-module/1.0.0> ;
<http://w3id.org/dice-research/enexa/module/extraction/parameter/urls_to_process> <IRI to file listing file urls to process>;
<http://w3id.org/dice-research/enexa/module/extraction/parameter/path_generation_parameters> <a json file of parameters for the module>.
```

The output file includes the KG in .ttl format

## Dice embeddings module

To initiate the DICE Embeddings module within the ENEXA service, submit the following request to the service endpoint '/start-container'. This module, focused on a hardware-agnostic framework for large-scale knowledge graph embeddings, provides a comprehensive guide on training and deploying knowledge graph embedding models.

### Goal

### Module Details

- **Module Instance Type:** enexa:ModuleInstance
- **Experiment:** <[experiment IRI]>
- **Algorithm Instance:** <http://w3id.org/dice-research/enexa/module/dice-embeddings/1.0.0>

### Parameters

The DICE Embeddings module requires the following parameters:

- **Batch Size:** {[batch size]}
- **Embedding Dimension:** {[]embeddings dimension}
- **Embedding Model:** <http://w3id.org/dice-research/enexa/module/dice-embeddings/algorithm/DistMult>
- **Number of Epochs:** {[number of epochs]}
- **Path to Knowledge Graph (KG):** <[]knowledge graph IRI>

### Execution
```ttl
@prefix alg: <http://www.w3id.org/dice-research/ontologies/algorithm/2023/06/> .
@prefix enexa:  <http://w3id.org/dice-research/enexa/ontology#> .
@prefix prov:   <http://www.w3.org/ns/prov#> .
@prefix hobbit: <http://w3id.org/hobbit/vocab#> . 
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .

[] rdf:type enexa:ModuleInstance ;
enexa:experiment <[experiment IRI]> ;
alg:instanceOf <http://w3id.org/dice-research/enexa/module/dice-embeddings/1.0.0> ;
<http://w3id.org/dice-research/enexa/module/dice-embeddings/parameter/batch_size> {[batch size]};
<http://w3id.org/dice-research/enexa/module/dice-embeddings/parameter/embedding_dim> {[]embeddings dimention};
<http://w3id.org/dice-research/enexa/module/dice-embeddings/parameter/model> <http://w3id.org/dice-research/enexa/module/dice-embeddings/algorithm/DistMult>;
<http://w3id.org/dice-research/enexa/module/dice-embeddings/parameter/num_epochs> {[number of epochs]};
<http://w3id.org/dice-research/enexa/module/dice-embeddings/parameter/path_single_kg> <[]knowledge graph IRI>.
```
### Additional Information

#### DICE Embeddings Framework

The DICE Embeddings framework serves as a hardware-agnostic solution for large-scale knowledge graph embeddings. It facilitates the training and deployment of knowledge graph embedding models, offering flexibility across various computing systems, from single CPUs to GPU clusters.

#### Usage Guidelines

This module provides users with a step-by-step guide on leveraging the DICE Embeddings framework. From setting the batch size to determining the embedding dimension and selecting the appropriate model, users can seamlessly integrate knowledge graph embeddings into their experiments.

#### Key Features

1. **Hardware-Agnostic:** DICE Embeddings is designed to be versatile, accommodating a range of computing systems for training and deploying knowledge graph embedding models.

2. **Scalability:** The framework supports large-scale knowledge graphs, making it suitable for projects with extensive data requirements.

3. **Pretrained Models:** The repository accompanying the module includes code, documentation, and pretrained models, expediting the integration process.

#### Getting Started

To get started with DICE Embeddings, send the provided module instance details to the '/start-container' endpoint, ensuring to replace placeholders with the appropriate experiment, batch size, embeddings dimension, number of epochs, and knowledge graph IRI. The module empowers users to harness the capabilities of a hardware-agnostic framework for large-scale knowledge graph embeddings.

[also visit the project repository page](https://github.com/dice-group/dice-embeddings) 

## Class Expression Learning (CEL)

CEL is a machine learning method specifically tailored for learning class expressions within knowledge graphs. In the realm of knowledge graphs, class expressions serve as descriptions of the properties of entities. For instance, a class expression could define all individuals residing in a specific city or all products manufactured by a particular company.

This module equips users with the capability to automatically learn complex class expressions from their knowledge graphs. By initiating the CEL Training module, users can harness machine learning techniques to derive meaningful insights and patterns from their data.

### Parameters
All parameters use the namespace `http://w3id.org/dice-research/enexa/module/cel-deploy/parameter/`.

- **endpoint**: A SPARQL endpoint hosting the knowledge graph on which the class expression learning is being executed.

### Execution 

The module provides a web service that takes a learning problem and provides class expressions as result. More details can be found in the [Ontolearn](https://github.com/dice-group/Ontolearn) project.

#### Serve
this module can serve a http endpoint and accept requests , for starting the service for this the bellow request should send  
```ttl
@prefix alg: <http://www.w3id.org/dice-research/ontologies/algorithm/2023/06/> .
        @prefix enexa:  <http://w3id.org/dice-research/enexa/ontology#> .
        @prefix prov:   <http://www.w3.org/ns/prov#> .
        @prefix hobbit: <http://w3id.org/hobbit/vocab#> . 
        @prefix rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
        @prefix rdfs:   <http://www.w3.org/2000/01/rdf-schema#> .

        [] rdf:type enexa:ModuleInstance ;
        enexa:experiment <[experimentIRI]> ;
        alg:instanceOf <http://w3id.org/dice-research/enexa/module/cel-deploy/1.0.0> ;
        <http://w3id.org/dice-research/enexa/module/cel-deploy/parameter/endpoint> <[SPARQL endpoint URL]>.
```

#### Web Service Endpoint
After starting the module, the endpoint can be accessed using the following method:
```
http://[container name]:7860/predict
```
The container name can be accessed from the ENEXA meta data graph.

The method takes the following JSON data
```json
{
        "pos": ["[list-of-positive-examples]","..."] ,
        "neg": ["[list-of-negative-examples]","..."],
        "model": "[name-of-the-learning-algorithm]",
        "max_runtime": "[maximum-runtime-in-seconds]",
        "iter_bound": "[maximum-iterations-during-training]",
        "path_to_pretrained_drill": "pretrained_drill",
        "path_embeddings": "[location-of-CSV-file-containing-the-embeddings]"
}
```

## TENTRIS module
### Goal

The goal of the TENTRIS module is to provide a fast and performant triple store.

### Module Details

The ENEXA module, instantiated with the Tentris RDF triple store, provides a comprehensive guide on leveraging Tentris—a robust, tensor-based RDF triple store. Tentris seamlessly integrates into the ENEXA service, offering efficient and high-performance capabilities for handling RDF data.

Tentris is specifically designed to handle RDF data using a tensor-based approach. This module elucidates the key aspects of Tentris, emphasizing its functionalities, optimal performance, and support for SPARQL queries.

#### Key Features

1. **Speed and Efficiency:** Tentris excels in speed, ensuring swift execution of RDF queries and operations. This feature contributes to the overall efficiency of ENEXA experiments.

2. **Tensor-Based Approach:** Tentris employs a tensor-based model for storing and processing RDF triples, enhancing scalability and facilitating complex data analysis.

3. **SPARQL Support:** Users can take advantage of Tentris's comprehensive support for SPARQL queries, enabling them to seamlessly interact with RDF data and retrieve relevant information.


### Parameters

### Execution 

To start the ENEXA module within the service, send the following request to the
```
/start-container
``` 
endpoint. Make sure that the specified file, indicated by the instance IRI in <[this should replace with the instance IRI which contains the file]>, has been previously added to the service using the 
```
/add-resource
```
endpoint. The initiation of execution involves linking the ENEXA module to a specific experiment identified by <[this should replace with the experimentIRI]>. The relevant RDF triples are detailed below: 
```ttl
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

### Additional Information
[TENTRIS Repository](https://github.com/dice-group/tentris)
