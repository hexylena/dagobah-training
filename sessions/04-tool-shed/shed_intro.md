layout: true
class: inverse, middle, large

---
class: special
# The Galaxy Tool Shed
intro to 'shed'

slides by @martenson, @mvdbeek

.footnote[\#usegalaxy / @galaxyproject]

---
class: larger

## Please Interrupt!

We like to answer your questions.

---

Tool Shed is to Galaxy as App Store is to iPhone.

It is a free service that hosts repositories containing Galaxy Tools.
Installing from the Tool Shed takes care of
  - dependencies
  - reference data tables
  - configuration files

---
Main Tool Shed (MTS) runs at http://toolshed.g2.bx.psu.edu and serves all Galaxies worldwide.

Everybody can create a repository.

Every repository is public including the whole history.

Local sheds can be run e.g. for private or custom-licensed tools.

???
We discourage running local TS.

---
## Vocabulary

* `wrapper` or `tool definition file` - The XML file that describes to Galaxy how the underlying software works, allowing Galaxy to render UI and execute the software in the right way.
--

* `repository` - A versioned code archive with tool(s) in Tool Shed.
--

* `revision` and `installable revision` - Every TS repo update generates a new revision but only certain (reproducibility-affecting) changes generate a new revision installable to Galaxy as a new version.

???
Every TS repo update can be propagated to the Galaxy though.

---
## Overview

* Tool Shed is a host (not a development) platform.
--

* Tool Development repository should be linked from the TS repository.
--

* Tool Shed allows administrators to pick any installable revision.
--

* Multiple installable revisions of any repository can be present in Galaxy.

---
![TS_big_picture.png](images/TS_big_picture.png)

---
class: normal
## Galaxy's Configuration

List of available sheds is defined in `tool_sheds_conf.xml` and Galaxy comes with the MTS enabled and the TTS disabled.
```xml
<?xml version="1.0"?>
<tool_sheds>
    <tool_shed name="Galaxy Main Tool Shed" url="https://toolshed.g2.bx.psu.edu/"/>
<!-- Test Tool Shed should be used only for testing purposes.
    <tool_shed name="Galaxy Test Tool Shed" url="https://testtoolshed.g2.bx.psu.edu/"/>
-->
</tool_sheds>
```

---
class: normal
## Simple TS repository (remove_beginning)

```shell
.
├── .shed.yml
├── remove_beginning.pl
├── remove_beginning.xml
└── test-data
    ├── 1.bed
    └── eq-removebeginning.dat
```

---
class: normal
## Simple TS repository (remove_beginning)

```shell
.
├── .shed.yml # metadata file
├── remove_beginning.pl # script that performs the requested
├── remove_beginning.xml # tool wrapper
└── test-data # this one is descriptive
    ├── 1.bed # test input file
    └── eq-removebeginning.dat # test output file
```

---
class: normal
### .shed.yml file

Contains tool's metadata.
```yml
categories:
- Text Manipulation
description: Remove lines from the beginning of a file.
long_description: |
  This tool removes the specified number of lines from the beginning
  of the input dataset.
name: remove_beginning
owner: devteam
remote_repository_url: https://github.com/galaxyproject/tools-devteam/tree/master/tools/remove_beginning
type: unrestricted
```

---
class: normal
## Complex TS Repository

```shell
.
├── .shed.yml
├── macros.xml
├── seqtk_comp.xml
├── seqtk_sample.xml
├── seqtk_seq.xml
├── seqtk_trimfq.xml
├── test-data
│   ├── seqtk_trimfq.fq
...
│   ├── seqtk_trimfq_be.fq
│   └── seqtk_trimfq_default.fq
└── tool_dependencies.xml
```

---
class: normal
## Complex TS Repository

```shell
.
├── .shed.yml
├── macros.xml # defines xml macros usable in other xml files
├── seqtk_comp.xml # wrapper for a tool
├── seqtk_sample.xml # wrapper for tool #2
├── seqtk_seq.xml # wrapper for tool #3
├── seqtk_trimfq.xml # wrapper for tool #4
├── test-data
│   ├── seqtk_trimfq.fq # bunch of test data
...
│   ├── seqtk_trimfq_be.fq
│   └── seqtk_trimfq_default.fq
└── tool_dependencies.xml # file that describes Galaxy how to get TS-supplied dependencies to fulfill the <requirements> in tool wrappers
```

---
### seqtk requirements

Requirements of the wrapper.
```xml
<requirements>
    <requirement type="package" version="1.2">seqtk</requirement>
</requirements>
```

Galaxy is aiming to be dependency resolution-agnostic.

---
# Intermezzo: Tool dependencies

To achieve the level of reproducibility Galaxy aims for it needs to be able to install *any tool at any version with the exact same dependencies at any time*.

Linux/MacOS package management is/was:
 - missing the scientific packages
 - avoiding or not maintaining old versions
 - unreliable
 - scattered

---
# Approach

* We aim to make Galaxy resolver-independent but Conda-oriented.
* What resolver is going to be used for the tool dependency is determined at runtime
and prioritised in the config file `dependency_resolvers_conf.xml`.
```xml
<dependency_resolvers>
  <tool_shed_packages />
  <galaxy_packages />
  <conda />
  <galaxy_packages versionless="true" />
  <conda versionless="true" />
<!-- other resolvers
  <tool_shed_tap />
  <homebrew />
-->
</dependency_resolvers>
```

---
class: smaller
### seqtk Conda package

```yml
package:
  name: seqtk
  version: 1.2
source:
  fn: v1.2.tar.gz
  url: https://github.com/lh3/seqtk/archive/v1.2.tar.gz
  md5: 255ffe05bf2f073dc57abcff97f11a37
build:
  number: 0
requirements:
  build:
    - gcc   # [not osx]
    - llvm  # [osx]
    - zlib
  run:
    - zlib
about:
  home: https://github.com/lh3/seqtk
  license: MIT License
  summary: Seqtk is a fast and lightweight tool for processing sequences in the FASTA or FASTQ format
test:
  commands:
    - seqtk seq
```

---
class: normal
### tool_dependencies.xml

A deprecated way that uses TS 'recipe'.

```xml
<?xml version="1.0"?>
<tool_dependency>
  <package name="seqtk" version="1.2">
    <repository name="package_seqtk_1_2" owner="iuc"/>
  </package>
</tool_dependency>
```

