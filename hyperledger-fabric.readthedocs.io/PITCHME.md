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
### Generate Network Artifacts

```sh
$ ./byfn.sh -m generate
```

> This first step generates all of the certificates and keys for our various network entities, the _genesis block_ used to bootstrap the ordering service, and a collection of configuration transactions required to configure a Channel

---
### Bring Up the Network

```sh
./byfn.sh -m up
```

```sh
# we use the -l flag to specify the chaincode language
# forgoing the -l flag will default to Golang
$ ./byfn.sh -m up -l node
```

---
### Bring Down the Network

```sh
$ ./byfn.sh -m down
```

---
### Crypto Generator

- cryptogen: used to generate the cryptographic material(x509 certs and signing keys)

These certificates are representative of identities, and they allow for sign/verify authentication to take place as our entities communicte and transact.

---
#### crypt-config.yaml

```yaml
OrdererOrgs:
  - Name: Orderer
    Domain: example.com
    Specs:
      - Hostname: orderer
PeerOrgs:
  - Name: Org1
    Domain: org1.example.com
    Template:
      Count: 2
    Users:
      Count: 1
  - Name: Org2
    Domain: org2.example.com
    Template:
      Count: 2
    Users:
      Count: 1
```

---
> After we run the _cryptogen_ tool, the generated certificates and keys will be saved to a folder titled _crypto-config_
> When ./byfn.sh -m up is executed, then cryptogen will be executed too. 
---
```
crypto-config
├── ordererOrganizations
│   └── example.com
│       ├── ca
│       ├── msp
│       ├── orderers
│       ├── tlsca
│       └── users
└── peerOrganizations
    ├── org1.example.com
    │   ├── ca
    │   ├── msp
    │   ├── peers
    │   ├── tlsca
    │   └── users
    └── org2.example.com
        ├── ca
        ├── msp
        ├── peers
        ├── tlsca
        └── users
```

---
#### Configuration Transaction Generator

The _configtxgen_ tool is used to create four configuration artifacts:
- orderer <span style="color:red">genesis block</span>
- channel <span style="color:red">configuration transaction</span>
- two <span style="color:red">anchor peer transactions</span> - one for each Peer Org.

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