```
cd ~/non-profit-blockchain/ngo-fabric
source fabric-exports.sh
source ~/peer-exports.sh 

cd ~/non-profit-blockchain
git pull origin master
cd ~
mkdir -p ./fabric-samples/chaincode/ngo
cp ./non-profit-blockchain/ngo-chaincode/src/* ./fabric-samples/chaincode/ngo

# Install
docker exec -e "CORE_PEER_TLS_ENABLED=true" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/home/managedblockchain-tls-chain.pem" \
    -e "CORE_PEER_LOCALMSPID=$MSP" -e "CORE_PEER_MSPCONFIGPATH=$MSP_PATH" -e "CORE_PEER_ADDRESS=$PEER"  \
    cli peer chaincode install -n ngo -l node -v v0 -p /opt/gopath/src/github.com/ngo

# Instantiate 
docker exec -e "CORE_PEER_TLS_ENABLED=true" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/home/managedblockchain-tls-chain.pem" \
    -e "CORE_PEER_LOCALMSPID=$MSP" -e "CORE_PEER_MSPCONFIGPATH=$MSP_PATH" -e "CORE_PEER_ADDRESS=$PEER"  \
    cli peer chaincode instantiate -o $ORDERER -C mychannel -n ngo -v v0 -c '{"Args":["init"]}' --cafile /opt/home/managedblockchain-tls-chain.pem --tls

# Upgrade
docker exec -e "CORE_PEER_TLS_ENABLED=true" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/home/managedblockchain-tls-chain.pem" \
    -e "CORE_PEER_LOCALMSPID=$MSP" -e "CORE_PEER_MSPCONFIGPATH=$MSP_PATH" -e "CORE_PEER_ADDRESS=$PEER"  \
    cli peer chaincode upgrade -o $ORDERER -C mychannel -n ngo -v v0 -c '{"Args":["init"]}' --cafile /opt/home/managedblockchain-tls-chain.pem --tls

# Query All
docker exec -e "CORE_PEER_TLS_ENABLED=true" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/home/managedblockchain-tls-chain.pem" \
    -e "CORE_PEER_ADDRESS=$PEER" -e "CORE_PEER_LOCALMSPID=$MSP" -e "CORE_PEER_MSPCONFIGPATH=$MSP_PATH" \
    cli peer chaincode query -C mychannel -n ngo -c '{"Args":["queryAllPatents"]}'

# Query
docker exec -e "CORE_PEER_TLS_ENABLED=true" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/home/managedblockchain-tls-chain.pem" \
    -e "CORE_PEER_ADDRESS=$PEER" -e "CORE_PEER_LOCALMSPID=$MSP" -e "CORE_PEER_MSPCONFIGPATH=$MSP_PATH" \
    cli peer chaincode query -C mychannel -n ngo -c '{"Args":["queryPatent","{\"applicationNumber\": \"US20160303254\"}"]}'

# Create
docker exec -e "CORE_PEER_TLS_ENABLED=true" -e "CORE_PEER_TLS_ROOTCERT_FILE=/opt/home/managedblockchain-tls-chain.pem" \
    -e "CORE_PEER_ADDRESS=$PEER" -e "CORE_PEER_LOCALMSPID=$MSP" -e "CORE_PEER_MSPCONFIGPATH=$MSP_PATH" \
    cli peer chaincode invoke -C mychannel -n ngo \
    -c  '{"Args":["createPatent","{\"applicationNumber\": \"US20160303254\", \"title\": \"Methylene carbamate linkers for use with targeted-drug conjugates\", \"applicant\": \"Seattle Genetics Inc\", \"inventors\": \"Robert KOLAKOWSKI, Scott Jeffrey, Patrick Burke\"}"]}' -o $ORDERER --cafile /opt/home/managedblockchain-tls-chain.pem --tls
```

```
cd ~/non-profit-blockchain/ngo-rest-api
nvm use lts/carbon
node app.js 

curl -s -X POST "http://localhost:3000/patents" -H "content-type: application/json" -d '{
    "applicationNumber":"US20110075897",
    "title":"Stain-based optimized compression of digital pathology slides",
    "applicant":"Genera Electric Co",
    "inventors":"Shai Dekel, Leon Google"
}'
```