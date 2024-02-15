---
title: Developing an ENEXA Module 
keywords: ENEXA Documentation
sidebar: mydoc_sidebar
toc: false
permalink: module_dev.html
folder: mydoc
---

Encapsulating your program as an ENEXA module is a good way to enable yourself or others to easily reuse your program within an AI pipeline. First, you should ensure that your machine fulfills the [prerequisites for running the ENEXA platform](quick_guide.html). In addition, we make use of [enexa-utils](enexa_utils.html) that need to be available on the machine.

In the following, we will go through a very simple example how an ENEXA module can be created. Further examples of modules can be found in the [list of ENEXA modules](modules_overview.html).

## Basic Example Module

This section will guide you through the structure of a typically module. The complete implementation that we are going to create is available on [github](https://github.com/EnexaProject/enexa-example-module). 

### 1. Module Funcionality

The module that we are going to create will receive a number, double it's value and return the new number within a file. We start by creating a file named [`example`](https://github.com/EnexaProject/enexa-example-module/blob/master/example) that contains a simple python program:
```python
#!/usr/bin/env python3
import sys

parameter = sys.argv[1]
result = int(parameter) * 2

with open('result.dat', 'w') as f:
    f.write(str(result))
```
If we have `python3` available, we can already run the program by calling:
```sh
./example 4
```
This will result in a file `result.dat` that contains the string `8`.

Note the following properties of our program:
1. It uses Python 3. So we have to ensure that the image that we are going to create comes with a Python 3 installed.
2. It takes an argument from the command line when calling it.
3. It produces a result file.

### 2. Starting the Meta Data File

Each ENEXA module needs to be described using a meta data file. The file contains general information about our module, including the parameters the program needs and the [docker image URN](metadata.html#image-identifiers). The meta data file has to be a [Turtle file](https://www.w3.org/TR/rdf11-primer/#section-turtle) containing the data as [RDF](https://www.w3.org/TR/rdf11-primer/). However, in most cases, no deeper knowledge about RDF is needed to define a module. We start by choosing a namespace for our module. Note that it is suggest to use an `http` namespace. You can choose any name you want. However, you should make sure to :
1. Not use a namespace that is already taken (e.g., the RDF namespace).
2. Not use the example namespace that we use throughout this example.

The first point avoids to reuse identifiers that already have a pre-defined meaning and could lead to problems later on. The second point should avoid that you may want to share a module just to see that a lot of other modules use the same identifiers.

For our example, we choose `http://example.org/example-module/` as namespace. Note that it ends with a slash so that we can add further elements to create identifiers (IRIs). For creating an IRI of our new module, we add it's version number. This will ensure that we can change the meta data easily later on, when publishing new versions. Since this will be the first version of our module, we name it `http://example.org/example-module/1.0.0`. With this information, we can already start to create our meta data file. We name it `module.ttl` an we write the following lines:
```turtle
@base           <http://example.org/example-module/> .
@prefix alg:    <http://www.w3id.org/dice-research/ontologies/algorithm/2023/06/> .
@prefix enexa:  <http://w3id.org/dice-research/enexa/ontology#> .
@prefix hobbit: <http://w3id.org/hobbit/vocab#> .
@prefix owl:    <http://www.w3.org/2002/07/owl#> .
@prefix rdf:    <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs:   <http://www.w3.org/2000/01/rdf-schema#> .

<1.0.0> a enexa:Module ;
	rdfs:label "My new example module"@en ;
	rdfs:comment "This is just a simple example module."@en ;
```
The first line defines the base namespace for the file. The next lines define prefixes, which help to keep the file readable. `<1.0.0>` represents our module (the complete IRI would be formed by adding the base namespace in front). We define that it is an ENEXA module and that it has a label and a description.

We need to add the input parameter and the result that our program creates. These also need identifiers. We use `input-number` for the parameter and `result-file` for the result, respectively (again, both will be HTTP IRIs when they are combined with our namespace). We add the following lines to our meta data file:
```turtle
	alg:parameter <input-number> ;
	alg:produces <result-file> .

<input-number> a alg:Parameter ;
	rdfs:label "Input number"@en ;
	rdfs:comment "The input number that should be doubled."@en .

<result-file> a alg:Result ;
	rdfs:label "Result file"@en ;
	rdfs:comment "The result file containing the doubled number."@en .
```
The first two lines still belong to the definition of our module and connect the parameter and result with the module. We define the parameter and the result and add labels and descriptions. The label and the description are not necessary but can ease the readability of the data in other applications.

### 3. Create a Docker Image

In this step, we want to encapsulate our program in to a Docker image in a way, that it can interact with ENEXA. To this end, we create a short shell script that contains
1. Reading the input parameters
2. Running our program, and
3. Storing the output file.

We name the script `module` and give it the following content:
```bash
#!/bin/sh
set -eu
./example $(enexa-parameter "http://example.org/example-module/input-number")
enexa-add-file result.dat "http://example.org/example-module/result-file"
```
The lines above have the following effect:
1. The first line is the shebang, defining that `sh` should be used to execute the commands. 
2. The second line defines that in case of missing environmental variables an error should be thrown. That can help while searching for errors because it avoids the access of undefined variables. 
3. The thrid line comprises two of the aforementioned three steps. First, it retrieves the value of the input parameter using the `enexa-parameter` of the enexa-utils library. Second, it executes our `example` program. 
4. The last line runs the `enexa-add-file` script that adds the result file as result of our program to the enexa meta data and moves it to the shared directory.

For creating the Docker Image, we write the following file named `Dockerfile` in which we define the build process:
```Dockerfile
FROM python:3.9
WORKDIR /app
ADD . ./
CMD ./module

# Add ENEXA utils.
COPY --from=hub.cs.upb.de/enexa/images/enexa-utils:1 / /.
```
We start with `python:3.9` as base image. We define the directory `/app` as our working directory before we copy everything from our current directory (in which our previously created files are located) into the work directory. The line `CMD ./module` defines that we want to execute the `module` script when the module is started. The last line copies necessary scripts from the enexa-utils image.

We have to define an identifier for the image while creating it. The image identifier typically includes the location from which it can be downloaded. So including the Docker registry of your choice can be important. In this example, we use the following image name:
```
hub.cs.upb.de/enexa/images/enexa-example-module:1.0.0
```
Please ensure to choose a name that fits to your module instead of copying the name above. Note that we have added a version tag at the end, which expresses that our image has version 1.0.0. With this image name in mind, we can run the following command on the command line (you have to replace the our image name with the name that you have chosen):
```sh
docker build -t hub.cs.upb.de/enexa/images/enexa-example-module:1.0.0 .
```
This should build our image.

### 4. Finalize the Meta Data File

After creating the docker image, we have to add the image's name to the meta data of our module. We can do that as follows:
```diff
 <1.0.0> a enexa:Module ;
	rdfs:label "My new example module"@en ;
	rdfs:comment "This is just a simple example module."@en ;
+	hobbit:image <urn:container:docker:image:hub.cs.upb.de/enexa/images/enexa-example-module:1.0.0> ;
	alg:parameter <input-number> ;
	alg:produces <result-file> .
```
Note that we add a prefix to the image name as described [here](metadata.html#image-identifiers).

### 5. Test the Module

#### Application-based Testing

The best way to test a module is to implement a small application that makes use of it. We provide a [tutorial](application_dev.html)) for writing an application, which can be easily adapted to be used for a newly developed module. This is especially useful for complex modules, which triggers the creation of additional modules, makes use of other functionalities of the ENEXA service or provides a service that should be used by an application.

#### Script-based Testing

The `enexa-utils` project provides a simple environment with a triple store and a mockup of the ENEXA service. This environment can be used for testing the implementation of a module if we adapt the `module` script that we defined in [Step 3](3.-create-a-docker-image).
```diff
#!/bin/sh
set -eu
+
+# If this is a test run (added for simple, local testing)
+if [ "${TEST_RUN:-false}" = true ]
+then  
+  # things which ENEXA is supposed to do
+  mkdir -p $ENEXA_WRITEABLE_DIRECTORY
+  echo "PREFIX enexa: <http://w3id.org/dice-research/enexa/ontology#> INSERT DATA {
+    GRAPH <$ENEXA_META_DATA_GRAPH> {
+      <$ENEXA_MODULE_INSTANCE_IRI> <http://example.org/example-module/input-number> '4' }}" \
+    |sparql-update "$ENEXA_META_DATA_ENDPOINT"
+fi
+
./example $(enexa-parameter "http://example.org/example-module/input-number")
enexa-add-file result.dat "http://example.org/example-module/result-file"
```
The performs two tasks that are typically done by the ENEXA service:
1. It creates the writeable directory, and
2. It adds a value for the parameter of our module to the meta data graph.

In our example, we add a `4` as input value. Hence, when running the module, we would expect a result file with an `8` as content. After building the Docker image of our module again (this is necessary since we changed the script), we run the following command within the [`enexa-utils`](enexa-utils.html) directory that we can clone [from github](https://github.com/EnexaProject/enexa-utils):
```sh
$ docker compose up -d
[+] Running 2/2
 ✔ Container enexa   Started                                               0.6s 
 ✔ Container fuseki  Started                                               0.6s 
```
It starts a mock up service and a Fuseki triple store. We can now run our module using the following commands:
```sh
docker run --rm \
	-v $(PWD)/test-shared-dir:/shared \
	-e ENEXA_EXPERIMENT_IRI=http://example.org/experiment1 \
	-e ENEXA_META_DATA_ENDPOINT=http://admin:admin@fuseki:3030/test \
	-e ENEXA_META_DATA_GRAPH=http://example.org/meta-data \
	-e ENEXA_MODULE_INSTANCE_DIRECTORY=/shared/experiment1/module1 \
	-e ENEXA_MODULE_INSTANCE_IRI=http://example.org/moduleinstance-$$(date +%s) \
	-e ENEXA_SERVICE_URL=http://enexa:36321/ \
	-e ENEXA_SHARED_DIRECTORY=/shared \
	-e ENEXA_WRITEABLE_DIRECTORY=/shared/experiment1 \
	-e TEST_RUN=true \
	--network enexa-utils_default \
	hub.cs.upb.de/enexa/images/enexa-example-module:1.0.0
```
Note that the command contains the address of the Fuseki instance as well as the enexa service mockup that we started before.
It also mounts a local directory called `test-shared-dir` as shared volume and sets the names of the shared, writable and module instance directory accordingly. The run of our module should print the following output to the command line:
```sh
Successfully send a SPARQL update
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> PREFIX sd: <http://www.w3.org/ns/sparql-service-description#> PREFIX enexa: <http://w3id.org/dice-research/enexa/ontology#> SELECT ?v WHERE { GRAPH <http://example.org/meta-data> { {<http://example.org/moduleinstance-1708017033> <http://example.org/example-module/input-number> ?l FILTER(isLiteral(?l)) BIND(str(?l) AS ?v)} UNION {<http://example.org/moduleinstance-1708017033> <http://example.org/example-module/input-number> [rdf:value ?v]} UNION {<http://example.org/moduleinstance-1708017033> <http://example.org/example-module/input-number> [sd:endpoint ?v]} UNION {<http://example.org/moduleinstance-1708017033> <http://example.org/example-module/input-number> [enexa:location ?l] BIND(REPLACE(?l, 'enexa-dir:/', '/shared') AS ?v)} }}
http://example.org/new-resource-iri-1708017034175784201
```
The last line would be the IRI of the newly created file in our meta data graph. However, the mockup service does not perform any inserts into the meta data graph. Hence, we check the shared file directory for the latest file that has been written. In our example, this is the following file:
```
$ cat test-shared-dir/experiment1/module1/UIQwmuDfxQ.result.dat
8
```
This is the expected output, which shows that our module seems to work as expected.
