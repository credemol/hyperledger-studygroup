Hyberledger Documents
===

---
# Resources

- [http://hyperledger-fabric.readthedocs.io/en/latest/index.html](http://hyperledger-fabric.readthedocs.io/en/latest/index.html)


# Getting Started

- Prerequisites
- Getting Started
- Hyperledger Fabric Samples

---
## Prerequisites (TODO)

---
## Getting Started (TODO)

---
## Hyperledger Fabric Samples

```sh
$ mkdir ~/Dev
$ cd ~/Dev
$ git clone -b master https://github.com/hyperledger/fabric-samples.git
$ cd fabric-samples
$ curl -sSL https://goo.gl/6wtTN5 | bash -s 1.1.0-preview

$ echo 'export PATH=$PATH:~/Dev/fabric-samples/bin' >> ~/.bash_profile
$ source ~/.bash_profile
```

---
# Key Concepts

- Introduction
- Hyperledger Fabric Capabilities
- Use Cases

---
## Introduction

---
## Hyperledger Fabric Capabilities

---
## Hyperledger Fabric Model

---
## Use Cases

---
# Tutorial

- Building Your First Network
- Writing Your First Application
- Reconfiguring Your First Network
- Chaincode Tutorials
- Chaincode for Developers
- Chaincode for Operators
- System Chaincode Plugins

---
## Building Your First Network

### Initialize

```sh
$ cd ~/Dev/fabric-samples
$ cd first-network

$ ./byfn.sh --help

```

---
#### Generate Network Artifacts

```sh
$ ./byfn.sh -m generate
```

> This first step generates all of the certificates and keys for our various network entities, the _genesis block_ used to bootstrap the ordering service, and a collection of configuration transactions required to configure a Channel
---
## Writing Your First Application

---
## Reconfiguring Your First Network

--- 
## Chaincode Tutorials

---
## Chaincode for Developers

---
## Chaincode for Operators

--- 
## System Chaincode Plugins