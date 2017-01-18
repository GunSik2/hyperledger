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
```

- golang installation
  - https://golang.org/dl/
- golang config (cygwin on windows env.)
```
export GOROOT=/cygdrive/c/Program Files (x86)/Go7
expert GOPATH=$GOPATH:/home/user/Workspace/blockchain
export PATH=$PATH:GOROOT/bin
```


### Validating Peer 구동
