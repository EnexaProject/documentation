---
title: Developing an ENEXA Application 
keywords: ENEXA Documentation
sidebar: mydoc_sidebar
toc: false
permalink: application_dev.html
folder: mydoc
---

This tutorial guides you through creating a basic Java application to interact with the ENEXA service. The application will perform tasks such as adding a file to the service using the "add_resource" endpoint and initiating a triple extraction module by sending a request with the IRI of the added file. Finally, it will display the result as a string.

Assumptions:

we assume the Enexa service hosted at http://localhost:8080

## Step 1 create a new experiment
The first step involves sending an HTTP POST request to "/start-experiment" as shown below:

```java
.
.
.
Model model = requestRDF(enexaURL + "/start-experiment", null);
.
.
.
```
The HTTP requests are handled using the requestRDF method defined as follows: 
```java
protected Model requestRDF(String url, Model data) {
        HttpPost request = new HttpPost(url);
        request.setHeader("Accept", "application/ld+json");
        request.setHeader("Content-type", "application/ld+json");
        if (data != null) {
            try (StringWriter writer = new StringWriter()) {
                data.write(writer, "JSON-LD");
                request.setEntity(new StringEntity(writer.toString()));
            } catch (IOException e) {
                LOGGER.error("Catched unexpected exception while adding data to the request. Returning null.", e);
                return null;
            }
        }
        Model model = null;
        try (CloseableHttpResponse httpResponse = client.execute(request)) {
            if (httpResponse.getCode() >= 300) {
                throw new IllegalStateException("Received HTTP response with code " + httpResponse.getCode());
            }

            try (InputStream is = httpResponse.getEntity().getContent()) {
                model = ModelFactory.createDefaultModel();
                model.read(is, "", "JSON-LD");
            }
        } catch (Exception e) {
            LOGGER.error("Caught an exception while running request. Returning null.");
            return null;
        }
        return model;
    }
```

The response to this request will be in JSON-LD format. An example response is:
```

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

Parsing the response can be done as shown:


```java

        Resource expResource = RdfHelper.getSubjectResource(model, RDF.type, ENEXA.Experiment);
        if (expResource == null) {
            throw new Exception("Couldn't find experiment resource.");
        }
        experimentIRI = expResource.getURI();
        LOGGER.info("Started an experiment: {}", experimentIRI);
        // Get meta data endpoint and graph
        Resource resource = RdfHelper.getObjectResource(model, expResource, ENEXA.metaDataEndpoint);
        if (resource == null) {
            throw new Exception("Couldn't find the experiment's meta data endpoint.");
        }
        metaDataEndpoint = resource.getURI();
        resource = RdfHelper.getObjectResource(model, expResource, ENEXA.metaDataGraph);
        if (resource == null) {
            throw new Exception("Couldn't find the experiment's meta data graph.");
        }
        metaDataGraph = resource.getURI();
        LOGGER.info("Meta data can be found at {} in graph {}", metaDataEndpoint, metaDataGraph);

```

## Step 2 add one file to service 

there is a file named "xx.json" like this 

```
[
"https://en.wikipedia.org/wiki/BASF"
]
```

To add a file named "xx.json" to the service, follow these steps:

Create a model with the metadata of the file.
Send the model to the service using the "add-resource" endpoint.
The process is illustrated in the Java code snippet below:

```java

        // Create a model with the meta data of our file
        Model fileDescription = ModelFactory.createDefaultModel();
        // The file itself will be a blank node
        String addFile = "@prefix enexa:  <http://w3id.org/dice-research/enexa/ontology#> .\n" +
            "  @prefix prov:   <http://www.w3.org/ns/prov#> .\n" +
            "\n" +
            "  [] a prov:Entity ; \n" +
            "      enexa:experiment <"+experimentIRI+"> ; \n" +
            "      enexa:location \""+ fileLocation +"\" .";
        fileDescription.read(new java.io.StringReader(addFile),null,"TURTLE");

        // Send the model
        Model response = requestRDF(enexaURL + "/add-resource", fileDescription);

        if (response == null) {
            throw new Exception("Couldn't add a resource to the meta data.");
        }

        // Get the new IRI of the resource
        Resource fileResource = RdfHelper.getSubjectResource(response, RDF.type,
            response.createResource("http://www.w3.org/ns/prov#Entity"));
        if (fileResource == null) {
            throw new Exception("Couldn't find the file resource.");
        }
        LOGGER.info("File resource {} has been created.", fileResource.getURI());
        jsonIri = fileResource.getURI();
        
```

Also, ensure that the "generation_parameters.json" file is added to the service.

```
{
    "max_length": 1024,
    "length_penalty": 0,
    "num_beams": 10,
    "num_return_sequences": 10,
    "return_dict_in_generate": true,
    "output_scores": true
}
```

## Step 3 start the module

for starting the module the request should sent to "start-container" end point 

```java

        Model instanceModel = ModelFactory.createDefaultModel();

        String start_module_message = "@prefix alg: <http://www.w3id.org/dice-research/ontologies/algorithm/2023/06/> .\n" +
            "@prefix enexa:  <http://w3id.org/dice-research/enexa/ontology#> .\n" +
            "@prefix prov:   <http://www.w3.org/ns/prov#> .\n" +
            "@prefix hobbit: <http://w3id.org/hobbit/vocab#> . \n" +
            "@prefix rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .\n" +
            "[] rdf:type enexa:ModuleInstance ; " +
            "enexa:experiment <"+experimentIRI+"> ; " +
            "alg:instanceOf <http://w3id.org/dice-research/enexa/module/extraction/1.0.0> ; " +
            "<http://w3id.org/dice-research/enexa/module/extraction/parameter/urls_to_process> <"+urlsIri+">;" +
            "<http://w3id.org/dice-research/enexa/module/extraction/parameter/path_generation_parameters> <"+jsonIri+">.";

        instanceModel.read(new java.io.StringReader(start_module_message), null, "TURTLE");

        Model response = requestRDF(enexaURL + "/start-container", instanceModel);

        if (response == null) {
            throw new Exception("Couldn't start a container.");
        }
        // Get the new IRI of the newly created module instance
        Resource instanceResource = RdfHelper.getSubjectResource(response, RDF.type, ENEXA.ModuleInstance);
        if (instanceResource == null) {
            throw new Exception("Couldn't find module instance resource.");
        }
        instanceIRI = instanceResource.getURI();
        
```

## Step 4 waiting to module finish 
