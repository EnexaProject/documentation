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

1. start the platform
2. create an experiment
3. start the module
4. Wait for the module to finish (poll during that)
5. 
