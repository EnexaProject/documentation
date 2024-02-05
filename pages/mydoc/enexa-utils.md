---
title: enexa-utils
keywords: ENEXA Documentation
sidebar: mydoc_sidebar
toc: false
permalink: enexa-utils.html
folder: mydoc
---

`enexa-utils` is a set of utilities that provide a simple way to retrieve parameters and store results for ENEXA modules.
All utilities expect to be running in the context of ENEXA module and rely on [ENEXA environment variables](parameters_env.html).

# Setup
In the `Dockerfile` of your module include the following:
```Dockerfile
COPY --from=enexa-utils:1 / /.
```

# Utilities

## enexa-parameter
Retrieve the value of the specified [module parameter](http://w3id.org/dice-research/enexa/ontology#Parameter) for the current module execution.

Usage:
- `./main.py --seed="$(enexa-parameter "http://example.org/parameter/seed")"`

## enexa-add-file
Store the specified existing file in ENEXA storage and create the corresponding experiment metadata.
If the file is already located in [`ENEXA_SHARED_DIRECTORY`](parameters_env.html) or its subdirectories (typically in `ENEXA_MODULE_INSTANCE_DIRECTORY`), it will be used directly without creating an additional copy.
An optional second argument makes the file to be the specified [module result](http://w3id.org/dice-research/enexa/ontology#Result) of the current module execution.

Usage:
- `enexa-add-file output.csv "http://example.org/result/output"`
- `enexa-add-file experiment.log`

# See also
<https://github.com/EnexaProject/enexa-utils>
