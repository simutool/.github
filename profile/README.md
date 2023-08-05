This repository contains the source code of the data management *software suite* developed for the Horizon 2020 EU project [SIMUTOOL](https://cordis.europa.eu/project/rcn/198371) (2015-2019). It was developed at the [University of Bamberg](https://www.uni-bamberg.de) in the [MOBI](https://www.uni-bamberg.de/en/mobi/) chair.

**Components**:

* KGService: Knowledge Graph that acts as a data catalog on top of a file system to store heterogenous data assets ([simutool/kgservice](https://github.com/simutool/kgservice)).
* Online Monitoring: An application for visualizing manufacturing sensor data and comparing it with reference data, as we well as uploading data assets and thier metadata to the KGService ([simutool/om-tool](https://github.com/simutool/om-tool)).
* Automatic Knowledge Uploade (AKU): End-user application to assist users to add and upload data assets and thier metadata to the KGService ([simutool/aku-client](https://github.com/simutool/aku-client)).
* Domain Model Creator: A plugin to load a domain model (domain semantics) into the KGService  ([simutool/model-builder ](https://github.com/simutool/model-builder)).
* Excel DM Reader: Convert a domain model from excel sheets to python dicts ([simutool/dm-reader](https://github.com/simutool/dm-reader)).
