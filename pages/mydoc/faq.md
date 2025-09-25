---
title: Frequently Asked Questions
keywords: ENEXA Documentation
sidebar: mydoc_sidebar
toc: false
permalink: faq.html
folder: mydoc
---



## About the ENEXA Project
**Question**: What is the ENEXA project?

**Answer**: ENEXA stands for "Efficient Explainable Learning on Knowledge Graphs." It's a European project focused on developing human-centered, explainable machine learning approaches for real-world knowledge graphs. The project is funded by the European Union under Horizon Europe.

## What is the focus of the ENEXA project?
**Question**: What is the focus of the ENEXA project?

**Answer**: The project focuses on knowledge graphs with rich semantics because of their growing popularity. It aims to develop scalable, transparent, and explainable hybrid machine learning algorithms that combine symbolic and sub-symbolic learning. The goal is to make machine learning on knowledge graphs practical for real-world applications, which is currently challenging due to the scale and inconsistency of these datasets.

## What is a key innovation of ENEXA?
**Question**: What is a key innovation of ENEXA?

**Answer**: A key innovation is its approach to explainability. Instead of just providing explanations, ENEXA focuses on a concept called "co-construction," where a human and the machine work together in a conversation to create human-understandable explanations. This approach is being deployed in three key sectors: business services, geospatial intelligence, and brand marketing.

## What is the motivation behind ENEXA's approach?
**Question**: What is the motivation behind ENEXA's approach?

**Answer**: While some current explainable machine learning approaches offer formal guarantees of completeness and correctness, they are often impractical to use on large, real-world knowledge graphs. The ENEXA project aims to overcome this by devising new methods that maintain these formal guarantees while being scalable enough to be deployed on real-world, messy data.

## Offline Platform

**Question**: Can I deploy the ENEXA platform completely offline without any internet access?

**Answer**: Yes, this is possible. However, it needs some additional preparation and a good knowledge about the images that will be used throughout experiments, since the platform won't be able to download Docker images.

The platform needs the following Docker images (They can also be gathered from the docker-compose file that is used to run the platform):
* `stain/jena-fuseki` (Any other triple store with a SPARQL 1.1 interface should be fine as well)
* `hub.cs.upb.de/enexa/images/enexa-service:latest`

The Docker images of the modules that should be used can be gathered from the module meta data files.

Images can be moved to an offline machine using the [`docker image save`](https://docs.docker.com/engine/reference/commandline/image_save/) and [`docker image load`](https://docs.docker.com/engine/reference/commandline/image_load/) commands.

## What is the main purpose of the ENEXA platform?
**Question**: What is the main purpose of the ENEXA platform?

**Answer**: The primary goal of the ENEXA platform is to make it easier to use the technical outcomes of the ENEXA project. It achieves this by connecting various software components through a system of data sharing and containerization, allowing different modules to work together seamlessly.

## How does the platform handle data and files?
**Question**: How does the platform handle data and files?**

**Answer**: The platform organizes all data as part of experiments. All the containers (modules and applications) within an experiment share a common directory, or shared volume, that contains all the experiment's data. There is also a central metadata store to provide rich information about the experiments, modules, and files.

##What is the role of containers in the platform?
**Question**: What is the role of containers in the platform?

**Answer**: The ENEXA platform uses containerization (with Docker as the underlying technology) to encapsulate every module and service. This approach ensures that you can combine various modules without worrying about their internal workings or dependencies, making the system highly flexible.

## What is the difference between the Application Layer and the Execution Layer?
**Question**: What is the difference between the Application Layer and the Execution Layer?

**Answer**: The Application Layer contains your main user application. This is the component that defines and manages an experiment, and it's typically the only part that directly interacts with the user. The Execution Layer is where the reusable modules (e.g., for data extraction or machine learning) are located. Your application uses the ENEXA service to execute these modules as needed.

## What is the ENEXA service, and what does it do?
**Question**: What is the ENEXA service, and what does it do?

**Answer**: The ENEXA service is a core component that provides essential functionalities to your application and the modules. It's responsible for starting and stopping modules, tracking the metadata and status of an experiment, and providing a central HTTP API for communication. It acts as the "controller" that brings all the different parts of the platform together.