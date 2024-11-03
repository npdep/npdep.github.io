# Server

[Start](../README.md) | [Architecture](./architecture.md)

- [Server](#server)
  - [Introduction](#introduction)
  - [Key Features](#key-features)
  - [Receiver Module and Lifecycle](#receiver-module-and-lifecycle)
  - [Options](#options)
  - [Receiver Module Structure](#receiver-module-structure)

## Introduction

The Network Protocol Data Exfiltration Project Server (npdep-server) is a powerful and flexible component of the NPDEP suite, designed to receive extract, and save exfiltrated data across multiple network protocols. As the counterpart to npdep-client, npdep-server enables security professionals to simulate realistic exfiltration events within a controlled environment, providing valuable insights into potential vulnerabilities in network defenses.

## Key Features


- **Customizable and Scalable Architecture:** With a modular, customizable setup, npdep-server can be adapted to handle various network configurations and integrate seamlessly with npdep-client. Users can modify parameters, such as accepted data formats and transfer methods, to reflect specific exfiltration use cases and security tests.

## Receiver Module and Lifecycle

To realize the core functions of the server, receiving, extracting and saving of data, npdep-server it is possible to implement custom receiver modules:

| Phase | Description |
 |---|---|
 | Creation |  The respective module is created. It receives the options specified for it. |
 | Start |   The respective module started. This allows further steps to be executed. |
 | Receive | The respective module receives incoming connections, processes data and ensures correct storage |
 | End | Receiving was completed. Final steps can be taken here. |


The special feature of the server modules here is the outsourcing of the receive phase to a separate thread. This is necessary as there may be different receive modules that would block each other without this outsourcing. If a new connection is established, it is received and processed in the newly created thread. Depending on the transmission and protocol, it may also be necessary to outsource the processing to a separate thread.

## Options

A JSON file is used to configure the server.

```json
{
    "options":{
        "modulePath": "" // By providing a path, you can 
                         // load modules not installed via pip
    },
    "modules": [
        {
            "name": "ReceiverModuleName",   
            "options": { // Pass the options of your choice to your sourcing module
                "port": 4567 // Example: Listen on port 4567
            }
        }
    ]
}
```

It is possible to have multiple receiver modules in one server configuration.

## Receiver Module Structure

A receiver module is implemented by a Python file in which a class is described. This class inherits from the Receiver class.

```py
import socket
from threading import *

from npdep_receiver.receiver.base.Receiver import Receiver

class CustomReceiverModule(Receiver):
    def __init__(self, options, logger):
        # Implements creation phase
        # logger is a custom object providing logging capabilities
        super().__init__("CustomReceiverModule", options, logger)
        self.socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.thread = Thread(target=self.handle)
        
    def start(self):
        # Implements start phase
        self.socket.bind((self.options["ip"], self.options["port"]))                  
        self.socket.listen(5)
        self.thread.start()
                
    def handle(self): 
        # Implements receiving phase 
        # See logging example
        self.logger.log(self.id + " started listening") 
        while self.isRunning:
            connection, address = self.socket.accept()
            self.logger.log(self.id + " accepted connection from: " + str(address)) 

    def end(self):
        # Implements end phase
```