# Data Exfiltration Protocol Specification

[Start](../README.md) | [Case Study](./case_study.md)

- [Data Exfiltration Protocol Specification](#data-exfiltration-protocol-specification)
  - [Introduction](#introduction)
  - [Header](#header)
  - [SystemInformationMessage](#systeminformationmessage)
  - [DataInformationMessage](#datainformationmessage)
  - [DataAcknowledgeMessage](#dataacknowledgemessage)
  - [DataContentMessage](#datacontentmessage)

## Introduction

Please read the [DEP Case Study](./case_study.md) before starting with this specification.

The Data Exfiltration Protocol is based on TCP and is divided into a header and a payload area.  The header area contains general information to identify the message and its length. The length of a protocol message is chosen so that it can be transported in a standard Ethernet frame with 1500 bytes.

## Header

**Message Structure**

| Name | Type | Size | Description |
|---|---|---|---|
| type | string | 1 Byte | Type of payload <br> 1 = SystemInformationMessage <br> 2 = DataInformationMessage <br> 3 = DataAcknowledgeMessage <br> 4 = DataContentMessage |
| length | int | 2 Byte | Length of payload |

**Description**

The header describes the `type` field, which identifies the DEP message contained in the payload area. Furthermore, the `length` field provides information on how many bytes can be found in the data area for the respective message.

The `type` and `length` information is therefore necessary in order to interpret the subsequent bytes correctly and to read the correct number of bytes from the network buffer in order to receive the message.


## SystemInformationMessage

**Payload Size:** 18 Byte

**Data Flow**

|From|To|
|---|---|
|Compromised System | Attacker System|

**Message Structure**

| Name | Type | Size | Description |
|---|---|---|---|
|systemId| string | 16 Byte | UUID of the system, from which data shall be exfiltrated |
| pkgSize | int | 2 Byte | Size in bytes into which the respective files to be exfiltrated are to be split. Maximum 1321 bytes|

**Description**

This type of message is used to identify the compromised system on the attacker side and to assign subsequently exfiltrated data. The UUID of the system in the `systemId` field is sent to the system controlled by the attacker. This UUID must be formed in accordance with RFC4122.

The `pkgSize` field describes the number of bytes into which the respective file to be exfiltrated is to be divided. This unit can be freely selected for each compromised system. This size must be transmitted to the server so that it knows how large the data units will be.


## DataInformationMessage

**Payload Size:** 316 Byte

**Data Flow**

|From|To|
|---|---|
|Compromised System | Attacker System |

**Message Structure**

| Name | Type | Size | Description |
|---|---|---|---|
|systemId| string | 16 Byte | UUID of the system, from which data shall be exfiltrated |
| sha256 | string | 32 Byte | SHA256 hash of data to be exfiltrated |
| size | int | 8 Byte | Size of data to be exfiltrated |
| path | string | 260 Byte | Path of the data to be exfiltrated on the compromised system |

**Description**

This message is used to transport the metadata of the file to be exfiltrated.

The `systemId` field describes the UUID that was sent to the exfiltration system by the SystemInformationMessage message. To uniquely identify the respective file, the SHA256 hash in the `sha256` field is generated and sent to the exfiltration system.

The respective file size in bytes is also transmitted in the `size` field. By knowing the respective file size in conjunction with the packet size information from the SystemInformationMessage message, it is possible to calculate the exact number of units into which the respective file is divided for transmission. With the field width of 8 bytes, individual files in the exabyte range can be sent.

The `path` field describes the path of the file on the compromised system. The path is specified here as 260 bytes, which refers to the MAX-PATH limit of Windows.

## DataAcknowledgeMessage

**Payload Size:** 49 Byte

**Data Flow**

|From|To|
|---|---|
|Attacker System | Compromised System |

**Message Structure**

| Name | Type | Size | Description |
|---|---|---|---|
|systemId| string | 16 Byte | UUID of the system, from which data shall be exfiltrated |
| sha256 | string | 32 Byte | SHA256 hash of data to be exfiltrated |
| dataState | int | 1 Byte | 0 = Data is not exfiltrated yet <br> 1 = Data is already exfiltrated <br> 2 = Data available under given path, but hash differs |

**Description**

Once the metadata of the file to be exfiltrated is available on the exfiltration system via the DataInformationMessage message, the status of the file to be exfiltrated can now be checked on the exfiltration system.

 It is evident that this message is sent from the exfiltration system to the compromised system and that the `systemId` and `sha256` fields enable the client on the compromised system to check that it has been received correctly and to uniquely identify the respective file using its SHA256 hash.

The `dataState` field provides information about the previously described exfiltration status of the respective file on the exfiltration system. After receiving this message, the software component on the compromised system can now decide whether the respective file should be exfiltrated or whether other measures should be taken.

## DataContentMessage

**Payload Size:** 57 - 1377 Byte

**Data Flow**

|From|To|
|---|---|
|Compromised System | Attacker System |

**Message Structure**

| Name | Type | Size | Description |
|---|---|---|---|
|systemId| string | 16 Byte | UUID of the system, from which data shall be exfiltrated |
| sha256 | string | 32 Byte | SHA256 hash of data to be exfiltrated |
| package | int | 8 Byte | Number of the package |
| data | bytes | 1 - 1321 Byte | data content|

**Description**

 After the metadata has been transferred by the DataInformationMessage message, the question now arises as to how the actual content of the respective file is to be transferred from the compromised system to the exfiltration system.

The `systemId` and `sha256` fields are used to send the UUID of the compromised system and the SHA256 hash of the respective file to the exfiltration system. This is important so that the file can be saved and assigned correctly on the exfiltration system. 

The `data` field contains the actual data of the respective file. If we consider the maximum size of a DEP packet within an Ethernet frame in an IP datagram as part of a TCP segment and assume the maximum size of the TCP/IP headers of 60 bytes each, it becomes clear that the data field can have a maximum size of 1321 bytes if it is to be sent in an Ethernet frame. 

Any number of bytes greater than 0 can be sent in the `data` field. This information is communicated beforehand for each system by the SystemInformationMessage message in the pkgSize field. As very few files are 1321 bytes or smaller, they must be sent in chunks. In this regard, the `package` field indicates which package the message in question is. On the exfiltration system side, the previously transmitted file and package size can also be used to calculate how many DataContentMessage messages must be sent for the respective file to be successfully transmitted.
