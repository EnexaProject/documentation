---
title: AI on demand integration
keywords: ENEXA Documentation
sidebar: mydoc_sidebar
toc: false
permalink: ai-on-demand.html
folder: mydoc
---


## AI on Demand Integration

Since the ENEXA modules are capable of performing AI tasks in a scalable, reproducible way, supporting their integration into AI-on-demand platforms (e.g., https://www.ai4europe.eu/) became an important requirement. In this article, we want to briefly describe how an ENEXA module can be published on the AI4EU marketplace according to the [documentation of the AI4EU project](https://github.com/ai4eu/tutorials/blob/master/Deliverable_AI4EU_D3.3_Platfrom_Manual.pdf).

Since AI-on-demand platforms come with their own orchestration implementation, an integration of the complete ENEXA platform into these platforms seems to be unnecessary. Instead, an integration of the single ENEXA modules is more promising. For the integration, i.e., the "on-boarding", of an ENEXA module into the AI4EU platform the steps of the [documentation]([documentation of the AI4EU project](https://github.com/ai4eu/tutorials/blob/master/Deliverable_AI4EU_D3.3_Platfrom_Manual.pdf)) should be followed. The integration of an ENEXA module provides the following advantages:
* The meta data of an ENEXA module comes with crucial data that is also needed by the AI4EU platform and, hence, can be reused.
    * A label and a description is typically part of the meta data of an ENEXA module. This meta data is also needed during the on-boarding process and can be copied and extended if needed.
    * The meta data also describes the API of an ENEXA module. This is needed for the Protobuf file that the AI4EU platform demands.
* The AI4EU platform builds upon the deployment of models using Kubernetes. ENEXA modules are provided as Docker images compatible with Kubernetes.

It should be noted that difference between the AI4EU platform and the ENEXA platform may lead to smaller adaptations of the existing ENEXA modules when used with AI4EU. For example, both platforms provide data sharing via the file system. However, the access to the provided files is organized differently.

