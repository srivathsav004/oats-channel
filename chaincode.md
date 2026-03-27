# OATS Traceability Chaincode Deployment Guide

This guide documents the complete deployment process for the OATS Traceability chaincode on your TRST01 network with `oatschannel`.

## Chaincode Overview

**Location**: `chaincode/oats-traceability-javascript/`

**Contract Name**: `OATSTraceabilityContract`

**Core Functions**:
- **Actor Management**: `CreateActor`, `GetActor`, `UpdateActorStatus`
- **Asset Management**: `DeclareAsset`, `GetAsset`, `UpdateAssetQuantity`
- **Facility Management**: `CreateFacility`, `GetFacility`
- **Traceability Events**: `CreateTraceabilityEvent`, `GetTraceabilityEvent`
- **Transformation**: `CreateTransformation`, `GetTransformation`
- **Compliance Documents**: `CreateComplianceDocument`, `GetComplianceDocument`
- **Query Functions**: `GetAssetHistory`, `GetAssetLineage`, `QueryAssetsByActor`, `QueryEventsByAsset`, `QueryEventsByType`, `QueryAssetsByCommodity`
- **Compliance & Audit**: `GetFullTraceabilityReport`, `GetComplianceStatus`

## 1) Preconditions

1. Network containers must be running (3 orderers + 2 TRST01 peers)
2. You must be in the oats-network directory:

```bash
cd ~/fabric-dev/fabric-samples/oats-network
```

3. Set Fabric CLI environment variables:

```bash
export PATH="$PWD/../bin:$PATH"
unset FABRIC_CFG_PATH
export FABRIC_CFG_PATH="$PWD/compose/docker/peercfg"

export CHANNEL_NAME=oatschannel
export CC_NAME=oats-traceability
export CC_VERSION=1.0
```

4. Set orderer TLS environment variables:

```bash
export ORDERER_CA="$PWD/organizations/ordererOrganizations/example.com/orderers/orderer1.example.com/tls/ca.crt"
export ORDERER_ADDRESS=localhost:7060
export ORDERER_TLS_HOST=orderer1.example.com

export CORE_PEER_TLS_ENABLED=true
```

## 2) Initial Deployment (SEQUENCE=1)

```bash
export SEQUENCE=1
```

### 2.1 Package the chaincode

```bash
peer lifecycle chaincode package ${CC_NAME}.tar.gz \
  --path "$PWD/chaincode/oats-traceability-javascript" \
  --lang node \
  --label ${CC_NAME}_${CC_VERSION}

export PACKAGE_ID=$(peer lifecycle chaincode calculatepackageid ${CC_NAME}.tar.gz)
echo "PACKAGE_ID=${PACKAGE_ID}"
```

### 2.2 Install on both TRST01 peers

```bash
# peer0.trst01 (7061)
export CORE_PEER_LOCALMSPID=TRST01MSP
export CORE_PEER_MSPCONFIGPATH="$PWD/organizations/peerOrganizations/trst01.example.com/users/Admin@trst01.example.com/msp"
export CORE_PEER_ADDRESS=localhost:7061
export CORE_PEER_TLS_ROOTCERT_FILE="$PWD/organizations/peerOrganizations/trst01.example.com/peers/peer0.trst01.example.com/tls/ca.crt"
peer lifecycle chaincode install ${CC_NAME}.tar.gz

# peer1.trst01 (8061)
export CORE_PEER_ADDRESS=localhost:8061
export CORE_PEER_TLS_ROOTCERT_FILE="$PWD/organizations/peerOrganizations/trst01.example.com/peers/peer1.trst01.example.com/tls/ca.crt"
peer lifecycle chaincode install ${CC_NAME}.tar.gz
```

### 2.3 Approve for TRST01 organization

```bash
export CORE_PEER_LOCALMSPID=TRST01MSP
export CORE_PEER_MSPCONFIGPATH="$PWD/organizations/peerOrganizations/trst01.example.com/users/Admin@trst01.example.com/msp"
export CORE_PEER_ADDRESS=localhost:7061
export CORE_PEER_TLS_ROOTCERT_FILE="$PWD/organizations/peerOrganizations/trst01.example.com/peers/peer0.trst01.example.com/tls/ca.crt"

peer lifecycle chaincode approveformyorg \
  -o ${ORDERER_ADDRESS} --ordererTLSHostnameOverride ${ORDERER_TLS_HOST} \
  --tls --cafile "${ORDERER_CA}" \
  --channelID ${CHANNEL_NAME} --name ${CC_NAME} --version ${CC_VERSION} \
  --package-id ${PACKAGE_ID} --sequence ${SEQUENCE}
```

### 2.4 Commit the chaincode definition

```bash
peer lifecycle chaincode commit \
  -o ${ORDERER_ADDRESS} --ordererTLSHostnameOverride ${ORDERER_TLS_HOST} \
  --tls --cafile "${ORDERER_CA}" \
  --channelID ${CHANNEL_NAME} --name ${CC_NAME} \
  --version ${CC_VERSION} --sequence ${SEQUENCE} \
  --peerAddresses localhost:7061 \
  --tlsRootCertFiles "$PWD/organizations/peerOrganizations/trst01.example.com/peers/peer0.trst01.example.com/tls/ca.crt" \
  --peerAddresses localhost:8061 \
  --tlsRootCertFiles "$PWD/organizations/peerOrganizations/trst01.example.com/peers/peer1.trst01.example.com/tls/ca.crt"
```

### 2.5 Verify deployment

```bash
peer lifecycle chaincode querycommitted --channelID ${CHANNEL_NAME} --name ${CC_NAME}
```

## 3) Test the Chaincode - Complete OATS Use Case

### 3.1 Create Actors (Farmers, Processors, etc.)

```bash
# Create Farmer Actor
peer chaincode invoke \
  -o ${ORDERER_ADDRESS} --ordererTLSHostnameOverride ${ORDERER_TLS_HOST} \
  --tls --cafile "${ORDERER_CA}" \
  -C ${CHANNEL_NAME} -n ${CC_NAME} \
  --peerAddresses localhost:7061 \
  --tlsRootCertFiles "$PWD/organizations/peerOrganizations/trst01.example.com/peers/peer0.trst01.example.com/tls/ca.crt" \
  -c '{"Args":["CreateActor","ACT-FARMER-001","Farmer","Green Valley Farms","FARM-REG-001","{\"phone\":\"+91-9876543210\",\"email\":\"farmer@greenvalley.com\"}","[\"CERT-ORG-001\",\"CERT-FAIR-001\"]"]}'

# Create Processor Actor
peer chaincode invoke \
  -o ${ORDERER_ADDRESS} --ordererTLSHostnameOverride ${ORDERER_TLS_HOST} \
  --tls --cafile "${ORDERER_CA}" \
  -C ${CHANNEL_NAME} -n ${CC_NAME} \
  --peerAddresses localhost:7061 \
  --tlsRootCertFiles "$PWD/organizations/peerOrganizations/trst01.example.com/peers/peer0.trst01.example.com/tls/ca.crt" \
  -c '{"Args":["CreateActor","ACT-PROCESSOR-001","Processor","Agri Processing Ltd","PROC-REG-001","{\"phone\":\"+91-9876543211\",\"email\":\"info@agriprocess.com\"}","[\"CERT-ISO-001\",\"CERT-HACCP-001\"]"]}'
```

### 3.2 Create Facility

```bash
peer chaincode invoke \
  -o ${ORDERER_ADDRESS} --ordererTLSHostnameOverride ${ORDERER_TLS_HOST} \
  --tls --cafile "${ORDERER_CA}" \
  -C ${CHANNEL_NAME} -n ${CC_NAME} \
  --peerAddresses localhost:7061 \
  --tlsRootCertFiles "$PWD/organizations/peerOrganizations/trst01.example.com/peers/peer0.trst01.example.com/tls/ca.crt" \
  -c '{"Args":["CreateFacility","FAC-PROC-001","Processor","ACT-PROCESSOR-001","{\"latitude\":19.7515,\"longitude\":75.7139,\"district\":\"Nashik\",\"state\":\"Maharashtra\",\"country\":\"India\"}","REG-FAC-001","50000","[\"CERT-ISO-001\",\"CERT-HACCP-001\"]"]}'
```

### 3.3 Declare Asset (Organic Cotton)

```bash
peer chaincode invoke \
  -o ${ORDERER_ADDRESS} --ordererTLSHostnameOverride ${ORDERER_TLS_HOST} \
  --tls --cafile "${ORDERER_CA}" \
  -C ${CHANNEL_NAME} -n ${CC_NAME} \
  --peerAddresses localhost:7061 \
  --tlsRootCertFiles "$PWD/organizations/peerOrganizations/trst01.example.com/peers/peer0.trst01.example.com/tls/ca.crt" \
  -c '{"Args":["DeclareAsset","AST-COTTON-001","Raw","COTTON-ORG","Organic Long Staple","1000","kg","ACT-FARMER-001","{\"latitude\":19.7515,\"longitude\":75.7139,\"district\":\"Nashik\",\"state\":\"Maharashtra\",\"country\":\"India\"}","2024-03-15","[\"CERT-ORG-001\",\"CERT-FAIR-001\"]"]}'
```

### 3.4 Create Traceability Events

```bash
# Declare Event
peer chaincode invoke \
  -o ${ORDERER_ADDRESS} --ordererTLSHostnameOverride ${ORDERER_TLS_HOST} \
  --tls --cafile "${ORDERER_CA}" \
  -C ${CHANNEL_NAME} -n ${CC_NAME} \
  --peerAddresses localhost:7061 \
  --tlsRootCertFiles "$PWD/organizations/peerOrganizations/trst01.example.com/peers/peer0.trst01.example.com/tls/ca.crt" \
  -c '{"Args":["CreateTraceabilityEvent","EVT-DEC-001","Declare","AST-COTTON-001","ACT-FARMER-001",null,null,"1000","kg","{\"season\":\"Kharif\",\"grade\":\"A\"}"]}'

# Request Event
peer chaincode invoke \
  -o ${ORDERER_ADDRESS} --ordererTLSHostnameOverride ${ORDERER_TLS_HOST} \
  --tls --cafile "${ORDERER_CA}" \
  -C ${CHANNEL_NAME} -n ${CC_NAME} \
  --peerAddresses localhost:7061 \
  --tlsRootCertFiles "$PWD/organizations/peerOrganizations/trst01.example.com/peers/peer0.trst01.example.com/tls/ca.crt" \
  -c '{"Args":["CreateTraceabilityEvent","EVT-REQ-001","Request","AST-COTTON-001","ACT-PROCESSOR-001","ACT-FARMER-001",null,"1000","kg","{\"price_per_kg\":150,\"delivery_date\":\"2024-03-20\"}"]}'

# Approve Event
peer chaincode invoke \
  -o ${ORDERER_ADDRESS} --ordererTLSHostnameOverride ${ORDERER_TLS_HOST} \
  --tls --cafile "${ORDERER_CA}" \
  -C ${CHANNEL_NAME} -n ${CC_NAME} \
  --peerAddresses localhost:7061 \
  --tlsRootCertFiles "$PWD/organizations/peerOrganizations/trst01.example.com/peers/peer0.trst01.example.com/tls/ca.crt" \
  -c '{"Args":["CreateTraceabilityEvent","EVT-APP-001","Approve","AST-COTTON-001","ACT-FARMER-001","ACT-PROCESSOR-001",null,"1000","kg","{\"approval_ref\":\"APR-2024-001\"}"]}'
```

### 3.5 Create Compliance Document

```bash
peer chaincode invoke \
  -o ${ORDERER_ADDRESS} --ordererTLSHostnameOverride ${ORDERER_TLS_HOST} \
  --tls --cafile "${ORDERER_CA}" \
  -C ${CHANNEL_NAME} -n ${CC_NAME} \
  --peerAddresses localhost:7061 \
  --tlsRootCertFiles "$PWD/organizations/peerOrganizations/trst01.example.com/peers/peer0.trst01.example.com/tls/ca.crt" \
  -c '{"Args":["CreateComplianceDocument","DOC-ORG-001","Certificate","Organic Certification Board","AST-COTTON-001","2024-01-01","2025-12-31","hash_abc123def456","ipfs://QmXxx..."]}'
```

### 3.6 Query Functions

```bash
# Get Asset Details
peer chaincode query -C ${CHANNEL_NAME} -n ${CC_NAME} \
  -c '{"Args":["GetAsset","AST-COTTON-001"]}'

# Get Asset History
peer chaincode query -C ${CHANNEL_NAME} -n ${CC_NAME} \
  -c '{"Args":["GetAssetHistory","AST-COTTON-001"]}'

# Get Asset Lineage
peer chaincode query -C ${CHANNEL_NAME} -n ${CC_NAME} \
  -c '{"Args":["GetAssetLineage","AST-COTTON-001"]}'

# Query Events by Asset
peer chaincode query -C ${CHANNEL_NAME} -n ${CC_NAME} \
  -c '{"Args":["QueryEventsByAsset","AST-COTTON-001"]}'

# Query Assets by Actor
peer chaincode query -C ${CHANNEL_NAME} -n ${CC_NAME} \
  -c '{"Args":["QueryAssetsByActor","ACT-FARMER-001"]}'

# Get Compliance Status
peer chaincode query -C ${CHANNEL_NAME} -n ${CC_NAME} \
  -c '{"Args":["GetComplianceStatus","AST-COTTON-001"]}'

# Get Full Traceability Report
peer chaincode query -C ${CHANNEL_NAME} -n ${CC_NAME} \
  -c '{"Args":["GetFullTraceabilityReport","AST-COTTON-001"]}'
```

## 4) Advanced Use Cases

### 4.1 Asset Transformation (Processing)

```bash
# Declare processed asset
peer chaincode invoke \
  -o ${ORDERER_ADDRESS} --ordererTLSHostnameOverride ${ORDERER_TLS_HOST} \
  --tls --cafile "${ORDERER_CA}" \
  -C ${CHANNEL_NAME} -n ${CC_NAME} \
  --peerAddresses localhost:7061 \
  --tlsRootCertFiles "$PWD/organizations/peerOrganizations/trst01.example.com/peers/peer0.trst01.example.com/tls/ca.crt" \
  -c '{"Args":["DeclareAsset","AST-COTTON-FIBER-001","Intermediate","COTTON-FIBER","Processed Cotton Fiber","950","kg","ACT-PROCESSOR-001","{\"latitude\":19.7515,\"longitude\":75.7139,\"district\":\"Nashik\",\"state\":\"Maharashtra\",\"country\":\"India\"}","2024-03-18","[\"CERT-ISO-001\"]"]}'

# Create transformation record
peer chaincode invoke \
  -o ${ORDERER_ADDRESS} --ordererTLSHostnameOverride ${ORDERER_TLS_HOST} \
  --tls --cafile "${ORDERER_CA}" \
  -C ${CHANNEL_NAME} -n ${CC_NAME} \
  --peerAddresses localhost:7061 \
  --tlsRootCertFiles "$PWD/organizations/peerOrganizations/trst01.example.com/peers/peer0.trst01.example.com/tls/ca.crt" \
  -c '{"Args":["CreateTransformation","TRANS-GIN-001","Ginning","[\"AST-COTTON-001\"]","[\"AST-COTTON-FIBER-001\"]","FAC-PROC-001","0.95"]}'

# Change of State event
peer chaincode invoke \
  -o ${ORDERER_ADDRESS} --ordererTLSHostnameOverride ${ORDERER_TLS_HOST} \
  --tls --cafile "${ORDERER_CA}" \
  -C ${CHANNEL_NAME} -n ${CC_NAME} \
  --peerAddresses localhost:7061 \
  --tlsRootCertFiles "$PWD/organizations/peerOrganizations/trst01.example.com/peers/peer0.trst01.example.com/tls/ca.crt" \
  -c '{"Args":["CreateTraceabilityEvent","EVT-COS-001","Change of State","AST-COTTON-001","ACT-FARMER-001","ACT-PROCESSOR-001","FAC-PROC-001","1000","kg","{\"process_type\":\"Ginning\",\"yield_ratio\":0.95}"]}'
```

## 5) Network Management

### 5.1 Stop/Start Network

**Stop containers (preserve ledger):**
```bash
docker-compose -f compose/compose-test-net.yaml -f compose/docker/docker-compose-test-net.yaml stop
```

**Start containers:**
```bash
docker-compose -f compose/compose-test-net.yaml -f compose/docker/docker-compose-test-net.yaml start
```

**Clean restart (delete ledger):**
```bash
docker-compose -f compose/compose-test-net.yaml -f compose/docker/docker-compose-test-net.yaml down -v
```

### 5.2 Chaincode Upgrade

When modifying chaincode:

```bash
# Increment sequence
export SEQUENCE=2

# Repackage
peer lifecycle chaincode package ${CC_NAME}.tar.gz \
  --path "$PWD/chaincode/oats-traceability-javascript" \
  --lang node \
  --label ${CC_NAME}_${CC_VERSION}

export PACKAGE_ID=$(peer lifecycle chaincode calculatepackageid ${CC_NAME}.tar.gz)

# Re-install on both peers
# (Repeat install commands from section 2.2)

# Re-approve for TRST01
# (Repeat approve command from section 2.3 with new PACKAGE_ID and SEQUENCE)

# Re-commit
# (Repeat commit command from section 2.4 with new SEQUENCE)
```

## 6) Troubleshooting

### 6.1 Common Issues

**"ProposalResponsePayloads do not match"**: Ensure deterministic timestamps using `ctx.stub.getTxTimestamp()`

**"Asset already exists"**: Check if asset ID was already used

**"Actor does not exist"**: Create actors before referencing them in events

**"Certificate not found"**: Verify TLS certificate paths are correct

### 6.2 Verification Commands

```bash
# Check chaincode status
peer lifecycle chaincode querycommitted --channelID ${CHANNEL_NAME} --name ${CC_NAME}

# Check installed chaincodes
peer lifecycle chaincode queryinstalled

# Check channel info
peer channel getinfo -c ${CHANNEL_NAME}
```

## 7) Integration Points

### 7.1 External Registry Integration

The chaincode supports integration with:
- **Farmer Registry**: Use `registry_reference` field in Actor schema
- **Land Registry**: Store land parcel IDs in `geo_reference` 
- **Facility Registry**: Use `registration_number` in Facility schema
- **Certification Registry**: Reference certification IDs in Asset and Document schemas

### 7.2 API Data Exchange Format

All transactions support JSON payloads following the OATS schema:

```json
{
  "event_type": "Dispatch",
  "asset_id": "AST-COTTON-001",
  "actor_from": "ACT-FARMER-001",
  "actor_to": "ACT-PROCESSOR-001",
  "quantity": 1000,
  "unit": "kg",
  "timestamp": "2024-03-18T10:22:00Z",
  "geo_reference": {
    "latitude": 19.7515,
    "longitude": 75.7139,
    "district": "Nashik",
    "state": "Maharashtra",
    "country": "India"
  }
}
```

Your OATS traceability network is now ready for production use with complete end-to-end supply chain tracking!
