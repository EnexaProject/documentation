---
title: Service Description
keywords: ENEXA Documentation
sidebar: mydoc_sidebar
toc: false
permalink: service_des.html
folder: mydoc
---

## Service Start

- The service needs to be started with a shared volume. The service has to be aware of the directory so that it can add it to
  - the meta data of experiments, and
  - newly created containers

- The service needs to be located in a virtual network and it needs to be aware of the network’s name so that it can add newly created containers to the same network.

- The service needs to be able to interact with the containerization software (e.g., Kubernetes)

- You can use the Postman collection to use api [collection](https://github.com/EnexaProject/enexa-service/blob/develop-ImplementTheUsageOfDirectories/ENEXA.postman_collection.json) 
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
| Response type| JSON-LD |
| Response content |-Experiment IRI <br/>-Meta data SPARQL endpoint URL <br/>-Shared directory path |
| Errors | ·	HTTP 500:	The service was not able to perform one of the necessary steps |

#### Example
request
```
POST /start-experiment HTTP/1.1
Content-Type: application/ld+json
Accept: application/ld+json
```

response 
```json lines

{
    "@id": "http://example.org/enexa/84b67db3-ee8f-4f47-9f81-25b3d34df09d",
    "http://w3id.org/dice-research/enexa/ontology#metaDataGraph": {
        "@id": "http://example.org/meta-data"
    },
    "http://w3id.org/dice-research/enexa/ontology#metaDataEndpoint": {
        "@id": "http://localhost:3030/mydataset"
    },
    "http://w3id.org/dice-research/enexa/ontology#sharedDirectory": "enexa-dir://app2/84b67db3-ee8f-4f47-9f81-25b3d34df09d",
    "@type": "http://w3id.org/dice-research/enexa/ontology#Experiment"
}

```

<hr>

| Name                | Meta data endpoint                                    |
|---------------------|-------------------------------------------------------|
| URL                 | /meta                                                 |
| Description         | This method returns the URL of a SPARQL endpoint and the IRI of the graph within this endpoint that contains the meta data for the experiment with the given IRI. |
| Type                | - HTTP GET                                            |
|                     | - synchronous                                         |
| Parameters          | - experimentIRI = Experiment IRI                          |
| Response type       | JSON-LD                                               |
| Response content    | - Meta data SPARQL endpoint URL                       |
|                     | - Meta data graph IRI (same response schema as /start-experiment) |
| Errors              | - HTTP 400:                                           |
|                     |   - Experiment IRI is not known / not available.      |
|                     | - HTTP 500:                                           |
|                     |   - There is no such SPARQL endpoint available.       |
#### Example
request
```
GET /meta?experimentIRI=http://example.org/enexa/84b67db3-ee8f-4f47-9f81-25b3d34df09d
```

response
```json lines
{
  "@id": "http://example.org/enexa/84b67db3-ee8f-4f47-9f81-25b3d34df09d",
  "http://w3id.org/dice-research/enexa/ontology#metaDataGraph": {
    "@id": "http://example.org/meta-data"
  },
  "http://w3id.org/dice-research/enexa/ontology#metaDataEndpoint": {
    "@id": "http://localhost:3030/mydataset"
  },
  "@type": "http://w3id.org/dice-research/enexa/ontology#Experiment"
}
```

<hr>

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
| Response type               | JSON-LD                                                       |
| Response content            | - Meta data of the newly created container                     |
| Errors                      | - HTTP 400:                                                   |
|                             |   o Experiment IRI is not known / not available.              |
|                             |   o The image does not exist or cannot be found.               |
|                             | - HTTP 500:                                                   |
|                             |   o An error occurs while communicating with the Kubernetes service. |

#### Example
request
```
POST /start-container HTTP/1.1
Content-Type: text/turtle
Accept: text/turtle

BODY :
  @prefix enexa:  <http://w3id.org/dice-research/enexa/ontology#> .
  @prefix prov:   <http://www.w3.org/ns/prov#> .

  [] a prov:Entity ; 
      enexa:experiment <> ; 
      enexa:location "/home/farshad/test/enexa/data.owl" .


@prefix alg: <http://www.w3id.org/dice-research/ontologies/algorithm/2023/06/> .
    @prefix enexa:  <http://w3id.org/dice-research/enexa/ontology#> .
    @prefix prov:   <http://www.w3.org/ns/prov#> .
    @prefix hobbit: <http://w3id.org/hobbit/vocab#> . 
    @prefix rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
    [] rdf:type enexa:ModuleInstance ;
    enexa:experiment <{{experimentIRI}}> ;
    alg:instanceOf <http://w3id.org/dice-research/enexa/module/tentris/0.2.0-SNAPSHOT-1> ;
    <http://w3id.org/dice-research/enexa/module/tentris/parameter/file> <{{ntFile}}>.
```

response
```
@prefix alg:    <http://www.w3id.org/dice-research/ontologies/algorithm/2023/06/> .
@prefix enexa:  <http://w3id.org/dice-research/enexa/ontology#> .
@prefix hobbit: <http://w3id.org/hobbit/vocab#> .
@prefix prov:   <http://www.w3.org/ns/prov#> .
@prefix rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .

<http://example.org/enexa/8e197678-7852-42ae-a9ec-1d05e1c90db0>
rdf:type             enexa:ModuleInstance ;
<http://w3id.org/dice-research/enexa/module/tentris/parameter/file>
<http://example.org/enexa/7a4dab1b-586c-46b9-a617-cb0db01a8c9c> ;
enexa:containerId    "66a3d6f6523a8521f7f8daa2957f06a833c865318d98d9491a43cc3adad04f37" ;
enexa:containerName  "enexa-1379311648" ;
enexa:experiment     <http://example.org/enexa/84b67db3-ee8f-4f47-9f81-25b3d34df09d> ;
alg:instanceOf       <http://w3id.org/dice-research/enexa/module/tentris/0.2.0-SNAPSHOT-1> .

```

<hr>

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
|                             | Additionally, plain resource IRI in "Content-Location" HTTP header |
| Response content            | - Meta data of the newly added file                             |
| Errors                      | - HTTP 400:                                                   |
|                             |   o Experiment IRI is not known / not available.              |
|                             |   o The resource URL does not exist or cannot be downloaded.  |
|                             | - HTTP 500:                                                   |
|                             |   o An error occurs while adding the resource.                |
#### Example
request
```
POST /add-resource HTTP/1.1
Content-Type: text/turtle
Accept: text/turtle

BODY :
  @prefix enexa:  <http://w3id.org/dice-research/enexa/ontology#> .
  @prefix prov:   <http://www.w3.org/ns/prov#> .

  [] a prov:Entity ; 
      enexa:experiment <http://example.org/enexa/84b67db3-ee8f-4f47-9f81-25b3d34df09d> ; 
      enexa:location "/data/example.owl" .
```

response
```
@prefix enexa: <http://w3id.org/dice-research/enexa/ontology#> .
@prefix prov:  <http://www.w3.org/ns/prov#> .

<http://example.org/enexa/6aed91d2-4ff1-426e-9c14-4572d2a8960e>
a                 prov:Entity ;
enexa:experiment  <http://example.org/enexa/84b67db3-ee8f-4f47-9f81-25b3d34df09d> ;
enexa:location    "/home/farshad/test/enexa/data.owl" .
```

<hr>

| Name                        | Get the status of a container                                                                                |
|-----------------------------|--------------------------------------------------------------------------------------------------------------|
| URL                         | /container-status                                                                                            |
| Description                 | This method returns status information for the given container that is gathered from the Kubernetes service. |
| Type                        | - HTTP POST                                                                                                  |
|                             | - synchronous                                                                                                |
| Parameters                  | - experiment = Experiment IRI                                                                                |
|                             | - container = Container IRI (or DNS name)                                                                    |
| Response type               | Some RDF serialization                                                                                       |
| Response content            | - The status of the container expressed as RDF. This could also express that the container does not exist.   |
| Errors                      | - HTTP 400:                                                                                                  |
|                             | o Experiment IRI is not known / not available.                                                               |
|                             | - HTTP 500:                                                                                                  |
|                             | o An error occurs while communicating with the Kubernetes service.                                           |

#### Example
request
```
POST /add-resource HTTP/1.1
Content-Type: application/json
Accept: text/turtle

BODY :
  {
    "moduleInstanceIRI":"http://example.org/enexa/832cf09e-b98f-40e8-b20c-fc1ecddc9424",
    "experimentIRI":"{{experimentIRI}}"
  }
```

response
```
<http://example.org/enexa/832cf09e-b98f-40e8-b20c-fc1ecddc9424>
        <http://w3id.org/dice-research/enexa/ontology#containerStatus>
                "running" ;
        <http://w3id.org/dice-research/enexa/ontology#experiment>
                <http://example.org/enexa/84b67db3-ee8f-4f47-9f81-25b3d34df09d> .
```

<hr>


| Name                        | Finish a container of the experiment                            |
|-----------------------------|---------------------------------------------------------------|
| URL                         | /stop-container                                           |
| Description                 | This method terminates a container that is part of the experiment. |
| Type                        | - HTTP POST                                                   |
|                             | - synchronous                                                  |
| Parameters                  | - experiment = Experiment IRI                                   |
|                             | - container IRI                                               |
| Parameter type              | multipart/form-data, RDF (e.g., JSON-LD)                        |
| Response type               | —                                                             |
| Response content            | HTTP 200                                                       |
| Errors                      | - HTTP 400:                                                   |
|                             |   o Experiment IRI is not known / not available.              |
|                             |   o Container IRI is not known / not available / does not belong to the given Experiment IRI |
|                             | - HTTP 500:                                                   |
|                             |   o An error occurs while communicating with the Kubernetes service. |

#### Example
request
```
POST /stop-container HTTP/1.1
Content-Type: text/turtle
Accept: text/turtle

BODY :
@prefix enexa:  <http://w3id.org/dice-research/enexa/ontology#> .
@prefix rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .

<http://example.org/enexa/832cf09e-b98f-40e8-b20c-fc1ecddc9424>
        rdf:type             enexa:ModuleInstance ;        
        enexa:containerId    "9a3e9c3c8b74631aad4f0ffd9c24eabfc923d7ac54d8b203363ddc0e29e36fe3" ;        
        enexa:experiment     <http://example.org/enexa/84b67db3-ee8f-4f47-9f81-25b3d34df09d>  .
```

response
```
<9a3e9c3c8b74631aad4f0ffd9c24eabfc923d7ac54d8b203363ddc0e29e36fe3>
        <http://w3id.org/dice-research/enexa/ontology#experiment>
                <http://example.org/enexa/84b67db3-ee8f-4f47-9f81-25b3d34df09d> .
```

<hr>

| Name                        | Finish an experiment                                           |
|-----------------------------|---------------------------------------------------------------|
| URL                         | /finish-experiment                                           |
| Description                 | This method finishes the experiment with the given IRI by stopping all its remaining containers. |
| Type                        | - HTTP POST                                                   |
|                             | - synchronous                                                  |
| Parameters                  | - experiment = Experiment IRI                                   |
| Parameter type              | multipart/form-data, RDF (e.g., JSON-LD)                        |
| Response type               | —                                                             |
| Response content            | HTTP 200                                                       |
| Errors                      | - HTTP 400:                                                   |
|                             |   o Experiment IRI is not known / not available.              |
|                             | - HTTP 500:                                                   |
|                             |   o An error occurs while communicating with the Kubernetes service. |

