- prerequiste
  - virtualbox
  - docker-machine

- docker vm creation
```
docker-machine create --driver virtualbox blockchain
eval $(docker-machine env blockchain)
```

- Hyperledger installation
```
docker pull hyperledger/fabric-baseimage:x86_64-0.2.2
docker pull hyperledger/fabric-membersrvc
docker pull hyperledger/fabric-peer

$ docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
hyperledger/fabric-baseimage    x86_64-0.2.2        sha256:4ac07        5 weeks ago         1.241 GB
hyperledger/fabric-baseimage    <none>              sha256:4ac07        5 weeks ago         1.241 GB
hyperledger/fabric-membersrvc   latest              sha256:b3654        3 months ago        1.417 GB
hyperledger/fabric-membersrvc   <none>              sha256:b3654        3 months ago        1.417 GB
hyperledger/fabric-peer         latest              sha256:21cb0        3 months ago        1.424 GB
hyperledger/fabric-peer         <none>              sha256:21cb0        3 months ago        1.424 GB

docker tag hyperledger/fabric-baseimage:x86_64-0.2.2 hyperledger/fabric-baseimage
$ docker images
REPOSITORY                      TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
hyperledger/fabric-baseimage    latest              sha256:4ac07        5 weeks ago         1.241 GB
hyperledger/fabric-baseimage    x86_64-0.2.2        sha256:4ac07        5 weeks ago         1.241 GB
hyperledger/fabric-baseimage    <none>              sha256:4ac07        5 weeks ago         1.241 GB
hyperledger/fabric-membersrvc   latest              sha256:b3654        3 months ago        1.417 GB
hyperledger/fabric-membersrvc   <none>              sha256:b3654        3 months ago        1.417 GB
hyperledger/fabric-peer         latest              sha256:21cb0        3 months ago        1.424 GB
hyperledger/fabric-peer         <none>              sha256:21cb0        3 months ago        1.424 GB
```

- golang installation:  https://golang.org/dl/
- golang config (cygwin on windows env.)
```
export GOROOT=/cygdrive/c/Program Files (x86)/Go7
expert GOPATH=$GOPATH:/home/user/Workspace/blockchain
export PATH=$PATH:GOROOT/bin
```
- docker-compose installation: https://github.com/docker/compose/releases

### Validating Peer 구동
- config
```
membersrvc:
  image: hyperledger/fabric-membersrvc
  ports:
    - "7054:7054"
  command: membersrvc
vp0:
  image: hyperledger/fabric-peer
  ports:
    - "7050:7050"
    - "7051:7051"
    - "7053:7053"
  environment:
    - CORE_PEER_ADDRESSAUTODETECT=true
    - CORE_LOGGING_LEVEL=DEBUG
    - CORE_PEER_ID=vp0
    - CORE_PEER_PKI_ECA_PADDR=membersrvc:7054
    - CORE_PEER_PKI_TCA_PADDR=membersrvc:7054
    - CORE_PEER_PKI_TLSCA_PADDR=membersrvc:7054
    - CORE_SECURITY_ENABLED=true
    - CORE_SECURITY_ENROLLID=test_vp0
    - CORE_SECURITY_ENROLLSECRET=MwYpmSRjupbT
  links:
    - membersrvc
  command: sh -c "sleep 5; peer node start --peer-chaincodedev"
```
- run
```
$ docker-compose.exe up
Creating blockchain_membersrvc_1
Creating blockchain_vp0_1
Attaching to blockchain_membersrvc_1, blockchain_vp0_1
vp0_1         | 06:09:52.518 [logging] LoggingInit -> DEBU 001 Setting default logging level to DEBUG for command 'node'
vp0_1         | 06:09:52.518 [nodeCmd] serve -> INFO 002 Running in chaincode development mode
vp0_1         | 06:09:52.518 [nodeCmd] serve -> INFO 003 Set consensus to NOOPS and user starts chaincode
vp0_1         | 06:09:52.518 [nodeCmd] serve -> INFO 004 Disable loading validity system chaincode
vp0_1         | 06:09:52.519 [peer] func1 -> INFO 005 Auto detected peer address: 172.17.0.3:7051
vp0_1         | 06:09:52.520 [peer] func1 -> INFO 006 Auto detected peer address: 172.17.0.3:7051
vp0_1         | 06:09:52.526 [eventhub_producer] AddEventType -> DEBU 007 registering BLOCK
vp0_1         | 06:09:52.526 [eventhub_producer] AddEventType -> DEBU 008 registering CHAINCODE
vp0_1         | 06:09:52.526 [eventhub_producer] AddEventType -> DEBU 009 registering REJECTION
vp0_1         | 06:09:52.526 [eventhub_producer] AddEventType -> DEBU 00a registering REGISTER
vp0_1         | 06:09:52.526 [nodeCmd] serve -> INFO 00b Security enabled status: true
vp0_1         | 06:09:52.526 [nodeCmd] serve -> INFO 00c Privacy enabled status: false
vp0_1         | 06:09:52.527 [db] open -> DEBU 00d Is db path [/var/hyperledger/production/db] empty [true]
vp0_1         | 06:09:52.530 [eventhub_producer] start -> INFO 00e event processor started
vp0_1         | 06:09:52.532 [db] open -> INFO 00f Setting rocksdb maxLogFileSize to 10485760
vp0_1         | 06:09:52.533 [db] open -> INFO 010 Setting rocksdb keepLogFileNum to 10
vp0_1         | 06:09:52.581 [nodeCmd] func1 -> DEBU 011 Registering validator with enroll ID: test_vp0

```
- stop
```
$ docker-compose.exe stop
```

# 개발 모드로 스마트 컨트랙 코드(체인코드) 작성

```
cd ~/Workspace/blockchain
mkdir -p github.com/hyperledger; cd github.com/hyperledger
git clone https://github.com/hyperledger/fabric.git
git checkout v0.6.1-preview
```
- 체인코드 빌드
```
cd $GOPATH/src/github.com/hyperledger/fabric/examples/chaincode/go/chaincode_example02
go build
```
-체인코드 실행
```
docker-machine ls // check ip
CORE_CHAINCODE_ID_NAME=mycc CORE_PEER_ADDRESS=192.168.99.100:7051 ./chaincode_example02
```
## REST API 를 통한 테스트
- 로그인
```
POST 192.168.99.100:7050/registrar
{
"enrollId": "admin",
"enrollSecret": "Xurw3yU9zI0l"
}

==> 
{
  "OK": "Login successful for user 'admin'."
}
```

- 체인코드 디플로이
```
POST 192.168.99.100:7050/chaincode
{
"jsonrpc": "2.0",
"method": "deploy",
"params": {
"type": 1,
"chaincodeID":{
"name": "mycc"
},
"ctorMsg": {
"args":["init", "a", "100", "b", "200"]
},
"secureContext": "admin"
},
"id": 1
}
==> 
{
  "jsonrpc": "2.0",
  "result": {
    "status": "OK",
    "message": "mycc"
  },
  "id": 1
}
```

- Invoke
  - a - 10 / b + 10
```
POST 192.168.99.100:7050/chaincode
{
"jsonrpc": "2.0",
"method": "invoke",
"params": {
"type": 1,
"chaincodeID":{
"name":"mycc"
},
"ctorMsg": {
"args":["invoke", "a", "b", "10"]
},
"secureContext": "admin"
},
"id": 3
}

==> 
{
  "jsonrpc": "2.0",
  "result": {
    "status": "OK",
    "message": "117ca387-6613-49ed-b324-d52031b36cef"
  },
  "id": 3
}
```

- Query
```
POST 192.168.99.100:7050/chaincode
{
"jsonrpc": "2.0",
"method": "query",
"params": {
"type": 1,
"chaincodeID":{
"name":"mycc"
},
"ctorMsg": {
"args":["query", "a"]
},
"secureContext": "admin"
},
"id": 5
}

==> 
{
  "jsonrpc": "2.0",
  "result": {
    "status": "OK",
    "message": "90"
  },
  "id": 5
}
```

- Another image creation to test with multiple nodes
```
$ console docker exec -it blockchain_membersrvc_1 bash
root@f772057996d8:/opt/gopath/src/github.com/hyperledger/fabric# vi membersrvc/membersrvc.yaml
root@f772057996d8:/opt/gopath/src/github.com/hyperledger/fabric# vi peer/core.yaml

$ docker commit blockchain_membersrvc_1 hyperledger/fabric-membersrvc:red
$ vi ~/Workspace/blockchain/docker-compose.yml
membersrvc:
  image: hyperledger/fabric-membersrvc:red
```

## 운영 모드로 스마트 컨트랙 코드(체인코드) 배치
