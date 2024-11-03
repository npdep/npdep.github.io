# Client

[Start](../README.md) | [Architecture](./architecture.md)

- [Client](#client)
  - [Introduction](#introduction)
  - [Key Features](#key-features)
  - [Module Types](#module-types)
  - [Module Lifecycle](#module-lifecycle)
  - [Module Interface](#module-interface)
  - [Options](#options)
  - [Sourcing Module Structure](#sourcing-module-structure)
  - [Transfer Module Structure](#transfer-module-structure)

## Introduction

The Network Protocol Data Exfiltration Project Client (npdep-client) is a modular tool designed to simulate data exfiltration scenarios across various network protocols. By replicating potential data leakage methods, npdep-client enables security professionals to test and enhance their detection and prevention strategies.

## Key Features

- **Modular Architecture:** npdep-client's design allows for easy integration and customization of sourcing and transfer modules, facilitating the simulation of diverse exfiltration techniques.

- **Protocol Flexibility:** With the modular architecture the client enables user to implement own data exfiltration szenarios with different network protocols, enabling them to explore various data exfiltration vectors and assess their security measures accordingly.

- **Customizable Configurations:** Users can tailor the module's behavior through configuration files, specifying parameters such as module paths, target IP addresses, and ports to align with specific testing requirements.

## Module Types

To realize the core functions of the client, the identification, collection and transfer of data, it has the following types of modules:

- **Sourcing:**
  Modules of this type map the skills of identifying and collecting the relevant data.

- **Transfer:**
  Modules of this type take data collected by the sourcing modules and transfer it to the npdep server on the exfiltration system under attacker control.

 This approach makes it possible to combine different sourcing modules with different transfer modules in order to be able to react to a wide variety of network configurations. For example, it would not only be possible to transfer local data on the hard disk of the compromised system, but also to search for data available via the network using a corresponding sourcing module.

## Module Lifecycle

 **Sourcing**

 The following table describes the phases of the life cycle of a sourcing module:

 | Phase | Description |
 |---|---|
 | Creation |  The respective module is created. It receives the options specified for it |
 | Initialization |  In this step, initial procedures can be carried out that are relevant for collecting the respective data. |
 | Sourcing |  In this step, the actual collection of the data is carried out. |
 | End | The collection was completed. Final steps can be taken here. |

**Transfer**

 The following table describes the phases of the life cycle of a transfer module:

 | Phase | Description |
 |---|---|
 | Creation |  The respective module is created. It receives the options specified for it |
 | Initialization |  In this step, initial procedures can be carried out that are relevant for transfering the respective data. |
 | Transfer |  In this step, the actual transmission of the data is carried out. The respective transfer module receives the collected data from the sourcing module for transmission to the exfiltration system. |
 | End | The collection was completed. Final steps can be taken here. |


## Module Interface

The following specification was created in order to be able to transfer data between the modules. It is required to distinguish between data that is already available as a file on the compromised system, for example, and data that can be queried remotely, e.g. from a domain controller or an unprotected REST API via the network.

```json
// Interface specification between sourcing and transfer modules
{
    "files": ["path/to/file.txt"],
    "data": ["see below"]
}
// Description of an object of the data list
{
    "name": "webserver_rest_api",
    "source": "192.168.1.1",
    "port": 80,
    "path": "path/to/file",
    "content": ["data1", "data2", "data3"]
}
```
Data already present on the system in the form of files is handed over to the transfer module as a path in the `files` list so that it can read and transfer the file. Remote data can be further described and transferred in the data list.

## Options

A JSON file is used to configure the client.

```json
{   
    "options":{
       "modulePath": "" // Optional: By providing a path, 
                        //           you can load modules from a custom location
    },
    "sourcing": {
        "module": {
            "name": "SourcingModuleName",
            "options": { } // Pass the options of your choice to your sourcing module
        }
    },
    "transfer": {
        "module": {
            "name": "TransferModuleName",
            "options": { } // Pass the options of your choice to your transfer module
        }
    }
}
```

Currently it is only possible to configure one sourcing module with an transfer module in an 1:1 relationship.

## Sourcing Module Structure

A sourcing module is implemented by a Python file in which a class is described. This class inherits from the Sourcing class.

```py
from npdep_common.interface.Interface import Interface

from npdep_sourcing.base.Sourcing import Sourcing

class CustomSourcingModule(Sourcing):
    def __init__(self, options, registration):
        # Implements creation phase
        # registration is a special object holding the generated uuid for
        # the compromised system in order to be able to identify the system
        # on server side. This uuid is generated at the first run of the client
        # and save in the current users home dir as a file called config.bin.
        # The uuid is accessible within all methods through self.registration.id
        super().__init__("CustomSourcingModule", options, registration)

    def init(self):
        # Implements initiation phase

    def get(self):
        # Implements sourcing phase
        # Interface.process creates the interface specification object
        # You are in charge of creating paths and data list according to 
        # the specification
        return Interface.process(files=paths, data=data)
    
    def end(self):
        # Implements end phase

```
## Transfer Module Structure

A transfer module is implemented by a Python file in which a class is described. This class inherits from the Transfer class.

```py

from npdep_transfer.base.Transfer import Transfer

class CustomTransferModule(Transfer):
    def __init__(self, options, registration) -> None:
        # Implements creation phase
        # registration is the same object described in sourcing module structure
        super().__init__("CustomTransferModule", options, registration)

    def init(self):
        # Implements initiation phase

    def send(self, container):
        # Implements transfer phase
        # container represents the interface specification object

    def end()
        # Implements end phase

```