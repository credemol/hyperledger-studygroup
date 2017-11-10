# Hyberledger e2e_cli example

End-to-End Flow 
## Resources
* [https://github.com/hyperledger/fabric/tree/master/examples/e2e_cli](https://github.com/hyperledger/fabric/tree/master/examples/e2e_cli)
* [https://github.com/hyperledger/fabric/blob/master/examples/e2e_cli/end-to-end.rst](https://github.com/hyperledger/fabric/blob/master/examples/e2e_cli/end-to-end.rst)

## Prerequisites

### Install git
```
$ sudo yum install git
```

### Install Docker

```
$ sudo yum-config-manager --enable ol7_addons
$ sudo yum install docker-engine

$ systemctl start docker
$ systemctl enable docker
$ systemctl status docker
```
### Enabling Non-root Users to Run Docker Commands
```
$ groupadd docker
$ service docker restart
$ sudo usermod -a -G docker holuser
```
>You must logout befere running docker command again

### Install Docker Compose
You can refer to [Install Compose](https://docs.docker.com/compose/install/#install-compose) 
```
$ sudo curl -L https://github.com/docker/compose/releases/download/1.17.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose
```

### Install Go 1.9 or later
```
$ cd /usr/local
$ sudo wget https://redirector.gvt1.com/edgedl/go/go1.9.2.linux-amd64.tar.gz
$ sudo tar xvf go1.9.2.linux-amd64.tar.gz

$ cd ~
$ mkdir go
$ vi ~/.bash_profile
```

In .bash_profile file, you need to set 3 environment variables as GOROOT, GOPATH and PATH

```shell
export GOROOT=/usr/local/go
export GOPATH=~/go
export PATH=$GOPATH/bin:$GOROOT/bin:$PATH
```

```
$ source ~/.bash_profile
$ go version
```

## Build Hyperledger fabric

### Bug fix
We can see following error message in the middle of the process of 'make docker'
```
Step 7/15 : RUN set -x     && cd /     && curl -fsSL "http://www.apache.org/dist/zookeeper/$DISTRO_NAME/$DISTRO_NAME.tar.gz" | tar -xz     && mv "$DISTRO_NAME/conf/"* "$ZOO_CONF_DIR"
 ---> Running in ef252cc0523b
+ cd /
+ curl -fsSL http://www.apache.org/dist/zookeeper/zookeeper-3.4.9/zookeeper-3.4.9.tar.gz
+ tar -xz
curl: (22) The requested URL returned error: 404 Not Found
```

You need to set one more environment vairable so that you can avoid this error.

```
$ find ./ -type f -exec grep -H 'DISTRO_NAME' {} \;
$ vi ./images/zookeeper/Dockerfile.in
#$ export DISTRO_NAME=zookeeper-3.4.10
#$ echo DISTRO_NAME
```
in line 31,
ARG DISTRO_NAME=zookeeper-3.4.9 --> zookeeper-3.4.10

You might see error messages below
```
d64 1.8.5-1.0.0+dfsg-4.5 [1566 kB]
Fetched 60.7 MB in 9min 35s (105 kB/s)
E: Failed to fetch http://archive.ubuntu.com/ubuntu/pool/main/e/erlang/erlang-runtime-tools_18.3-dfsg-1ubuntu3_amd64.deb  502  cannotconnect [IP: 91.189.88.162 80]

E: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?
The command '/bin/sh -c apt-get update -y && apt-get install -y --no-install-recommends     ca-certificates     curl     erlang-nox     erlang-reltool     haproxy     libicu5.     libmozjs185-1.0     openssl     cmake     apt-transport-https     gcc     g++     erlang-dev     libcurl4-openssl-dev     libicu-dev     libmozjs185-dev     make   && rm -rf /var/lib/apt/lists/*' returned a non-zero code: 100
make: *** [build/image/couchdb/.dummy-x86_64-1.0.5-snapshot-c7c8827] Error 100
```

To fix this, install required tools below

```
$ sudo yum install ca-certificates     curl     erlang-nox     erlang-reltool     haproxy     libicu5.     libmozjs185-1.0     openssl     cmake     apt-transport-https     gcc     g++     erlang-dev     libcurl4-openssl-dev     libicu-dev     libmozjs185-dev     make
```



Above is not covered in the web site.
```
$ mkdir -p $GOPATH/src/github.com/hyperledger
$ cd $GOPATH/src/github.com/hyperledger

$ git clone https://github.com/hyperledger/fabric.git

$ cd fabric
$ make release
$ make docker
```

> you can run *_git clone http://gerrit.hyperledger.org/r/fabric_* alternatively.


## Configuration Transaction Generator


```
$ cd examples/e2e_cli
$ ./generateArtifacts.sh my-channel
``` 

my-channel is the <CHANNEL-ID>


### Manually generate the artifacts(Optional)
> This session is now working. it will be written later.

```
$ os_arch=$(echo "$(uname -s)-amd64" | awk '{ print tolower($0)}')

$ echo $os_arch

$ ./../../release/$os_arch/bin/cryptogen generate --config=./crypto-config.yaml
```


### Run the end-to-end test with Docker
* CHANNEL_NAME: the same value that you specified above. in our case, it is my-channel
* TIMEOUT: Timeout, it is optional

```
$ CHANNEL_NAME=my-channel TIMEOUT=1000 docker-compose -f docker-compose-cli.yaml up -d
```

Wait, 60 seconds or so. Behind the scenes, there are transactions being sent to the peers. Execute a _docker image ls_ to view your active containers. You should see an output identical to the following:

```
$ docker logs -f cli

$ docker logs dev-peer0.org2.example.com-mycc-1.0
$ docker logs dev-peer0.org1.example.com-mycc-1.0
$ docker logs dev-peer1.org2.example.com-mycc-1.0
```

### Manually exercise the commands

```
$ docker container rm -f $(docker container ls -qa)

$ docker image ls dev-peer*
$ docker image rm $(docker image ls dev-peer* -q)

$ ./generateArtifacts.sh my-channel
```

#### Modify the docker-compose-cli file

```
$ vi docker-compose-cli.yaml
```
In line number 57, place a # to the left of the command. for example.

```yaml
#     command: /bin/bash -c './scripts/script.sh ${CHANNEL_NAME}; sleep $TIMEOUT'
```

Now restart your network. We will use *'my-channel'* as the <CHANNEL-ID>

```sh
$ CHANNEL_NAME="my-channel" TIMEOUT=1000 docker-compose -f docker-compose-cli.yaml up -d
```
#### Create another channel
We need four enviroment variables given below. These variables for peer0.org1.example.com are baked into the CLI container, therefore we can operate without passing them
```sh
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
CORE_PEER_ADDRESS=peer0.org1.example.com:7051
CORE_PEER_LOCALMSPID="Org1MSP"
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
```

Exec into the cli container
```sh
$ docker exec -it cli bash
root@0d78bb69300d:/opt/gopath/src/github.com/hyperledger/fabric/peer#
root@0d78bb69300d:/opt/gopath/src/github.com/hyperledger/fabric/peer# echo $CORE_PEER_MSPCONFIGPATH
root@0d78bb69300d:/opt/gopath/src/github.com/hyperledger/fabric/peer# echo $CORE_PEER_ADDRESS
root@0d78bb69300d:/opt/gopath/src/github.com/hyperledger/fabric/peer# echo $CORE_PEER_LOCALMSPID
root@0d78bb69300d:/opt/gopath/src/github.com/hyperledger/fabric/peer# echo $CORE_PEER_TLS_ROOTCERT_FILE

```

Create a channel with channel id as 'my-channel'

> *_Caution_*: The command below won't work properly. 
```sh
$ peer channel create -o orderer.example.com:7050 -c my-channel -f ./channel-artifacts/channel.tx --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/cacerts/ca.example.com-cert.pem
```

> You should run it like below
```sh
$ peer channel create -o orderer.example.com:7050 -c my-channel -f ./channel-artifacts/channel.tx --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```
refer to scripts/script.sh, look at createChannel() part of the file.

>ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem


#### Join Channel

```sh
$ peer channel join -b my-channel.block
```

#### Install chaincode onto a remote peer

```sh
$ peer chaincode install -n mycc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02
```

#### Instantiate chaincode and define the endorsement policy

> *_Caution_*: The command below won't work properly. 
```sh
$ peer chaincode instantiate -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/cacerts/ca.example.com-cert.pem -C my-channel -n mycc -v 1.0 -p github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org1MSP.member','Org2MSP.member')"
```

> You should run it like below
```sh
$ peer chaincode instantiate -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C my-channel -n mycc -v 1.0 -c github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org1MSP.member','Org2MSP.member')"
```

> peer chaincode instantiate -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "OR	('Org1MSP.member','Org2MSP.member')" 


#### Invoke chaincode

> *_Caution_*: The command below won't work properly. 
```sh
$ peer chaincode invoke -o orderer.example.com:7050 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/cacerts/ca.example.com-cert.pem  -C my-channel -n mycc -c '{"Args":["invoke","a","b","10"]}'
```

> You should run it like below

```sh
$ peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem  -C my-channel -n mycc -c '{"Args":["invoke","a","b","10"]}'
```

> peer chaincode invoke -o orderer.example.com:7050  --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C $CHANNEL_NAME -n mycc -c '{"Args":["invoke","a","b","10"]}' 

#### Query chaincode

```sb
peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
```