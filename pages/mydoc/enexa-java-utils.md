---
title: Meta Data
keywords: ENEXA Documentation
sidebar: mydoc_sidebar
toc: false
permalink: enexa-java-utils.html
folder: mydoc
---

# Enexa Java Utils

The `EnexaPathUtils` class, part of the Enexa Project, serves as a valuable utility for translating file paths between the ENEXA metadata graph and local file systems. This Java utility simplifies the process of converting paths, providing a seamless integration between ENEXA-specific path representations and their corresponding local file system equivalents.

## Key Features

1. **Path Translation:** Translate paths from the ENEXA metadata graph to their local representations and vice versa.
2. **Shared Directory Prefix:** Recognize and handle paths with the specified shared directory prefix, enabling effective translation.
3. **Error Handling:** Log errors if a local path is not within the specified shared directory, ensuring data integrity.

## Usage

- `translateEnexa2LocalPath(String enexaPath, String sharedDir)`: Translate ENEXA path to local path.
- `translateLocal2EnexaPath(File localFile, String sharedDir)`: Translate local file to ENEXA path.
- `translateLocal2EnexaPath(String localPath, String sharedDir)`: Translate local path to ENEXA path.

## Example

```java
String enexaPath = "enexa-dir://example/path";
String sharedDir = "/shared/directory/";
String localPath = EnexaPathUtils.translateEnexa2LocalPath(enexaPath, sharedDir);
System.out.println("Translated Local Path: " + localPath);
```

Note: Ensure that the shared directory is correctly specified to guarantee accurate path translations.



