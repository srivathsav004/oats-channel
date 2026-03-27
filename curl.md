# OATS Traceability API - Curl Examples

This document contains curl commands to test all the OATS Traceability Backend APIs.

## Base URL
```
http://localhost:3000
```

## Health Check

### Health Check
```bash
curl -X GET http://localhost:3000/health \
  -H "Content-Type: application/json"
```

## Actor Management APIs

### Create Actor
```bash
curl -X POST http://localhost:3000/actors/create \
  -H "Content-Type: application/json" \
  -d '{
    "actorId": "ACTOR001",
    "actorType": "FARMER",
    "name": "John Doe Farm",
    "registryReference": "REG123456",
    "contactDetails": {
      "email": "john@farm.com",
      "phone": "+1234567890",
      "address": "123 Farm Road, Agriculture City"
    },
    "certificationIds": ["CERT001", "CERT002"]
  }'
```

### Get Actor
```bash
curl -X GET http://localhost:3000/actors/ACTOR001 \
  -H "Content-Type: application/json"
```

### Update Actor Status
```bash
curl -X PUT http://localhost:3000/actors/ACTOR001/status \
  -H "Content-Type: application/json" \
  -d '{
    "newStatus": "Inactive"
  }'
```

## Asset Management APIs

### Declare Asset
```bash
curl -X POST http://localhost:3000/assets/declare \
  -H "Content-Type: application/json" \
  -d '{
    "assetId": "ASSET001",
    "assetType": "ORGANIC_WHEAT",
    "commodityCode": "WHT001",
    "variety": "Hard Red Winter",
    "quantity": 1000,
    "unit": "kg",
    "originActorId": "ACTOR001",
    "originLocation": {
      "latitude": 40.7128,
      "longitude": -74.0060,
      "address": "Farm Field A, Agriculture City"
    },
    "harvestDate": "2024-03-15",
    "certificationReference": ["CERT001"]
  }'
```

### Get Asset
```bash
curl -X GET http://localhost:3000/assets/ASSET001 \
  -H "Content-Type: application/json"
```

### Update Asset Quantity
```bash
curl -X PUT http://localhost:3000/assets/ASSET001/quantity \
  -H "Content-Type: application/json" \
  -d '{
    "newQuantity": 950,
    "reason": "Quality inspection loss"
  }'
```

## Facility Management APIs

### Create Facility
```bash
curl -X POST http://localhost:3000/facilities/create \
  -H "Content-Type: application/json" \
  -d '{
    "facilityId": "FAC001",
    "facilityType": "WAREHOUSE",
    "ownerActorId": "ACTOR001",
    "location": {
      "latitude": 40.7128,
      "longitude": -74.0060,
      "address": "123 Storage Ave, Logistics City"
    },
    "registrationNumber": "FAC_REG001",
    "capacity": 50000,
    "complianceCertificates": ["FAC_CERT001"]
  }'
```

### Get Facility
```bash
curl -X GET http://localhost:3000/facilities/FAC001 \
  -H "Content-Type: application/json"
```

## Traceability Event APIs

### Create Traceability Event
```bash
curl -X POST http://localhost:3000/events/create \
  -H "Content-Type: application/json" \
  -d '{
    "eventId": "EVENT001",
    "eventType": "Dispatch",
    "assetId": "ASSET001",
    "actorFrom": "ACTOR001",
    "actorTo": "ACTOR002",
    "facilityId": "FAC001",
    "quantity": 500,
    "unit": "kg",
    "eventMetadata": {
      "temperature": 22,
      "humidity": 45,
      "transportVehicle": "TRUCK001",
      "driver": "Driver Name"
    }
  }'
```

### Get Traceability Event
```bash
curl -X GET http://localhost:3000/events/EVENT001 \
  -H "Content-Type: application/json"
```

## Transformation APIs

### Create Transformation
```bash
curl -X POST http://localhost:3000/transformations/create \
  -H "Content-Type: application/json" \
  -d '{
    "processId": "PROCESS001",
    "processType": "MILLING",
    "inputAssets": ["ASSET001"],
    "outputAssets": ["ASSET002"],
    "facilityId": "FAC001",
    "yieldRatio": 0.85
  }'
```

### Get Transformation
```bash
curl -X GET http://localhost:3000/transformations/PROCESS001 \
  -H "Content-Type: application/json"
```

## Compliance Document APIs

### Create Compliance Document
```bash
curl -X POST http://localhost:3000/documents/create \
  -H "Content-Type: application/json" \
  -d '{
    "documentId": "DOC001",
    "documentType": "ORGANIC_CERTIFICATE",
    "issuingAuthority": "USDA Organic",
    "relatedAsset": "ASSET001",
    "validFrom": "2024-01-01",
    "validTo": "2025-01-01",
    "digitalHash": "sha256:abc123def456...",
    "storageUri": "ipfs://QmXxx..."
  }'
```

### Get Compliance Document
```bash
curl -X GET http://localhost:3000/documents/DOC001 \
  -H "Content-Type: application/json"
```

## Query Function APIs

### Get Asset History
```bash
curl -X GET http://localhost:3000/query/asset/ASSET001/history \
  -H "Content-Type: application/json"
```

### Get Asset Lineage
```bash
curl -X GET http://localhost:3000/query/asset/ASSET001/lineage \
  -H "Content-Type: application/json"
```

### Query Assets by Actor
```bash
curl -X GET http://localhost:3000/query/assets/actor/ACTOR001 \
  -H "Content-Type: application/json"
```

### Query Events by Asset
```bash
curl -X GET http://localhost:3000/query/events/asset/ASSET001 \
  -H "Content-Type: application/json"
```

### Query Events by Type
```bash
curl -X GET http://localhost:3000/query/events/type/Dispatch \
  -H "Content-Type: application/json"
```

### Query Assets by Commodity
```bash
curl -X GET http://localhost:3000/query/assets/commodity/WHT001 \
  -H "Content-Type: application/json"
```

## Compliance and Audit APIs

### Get Full Traceability Report
```bash
curl -X GET http://localhost:3000/compliance/report/ASSET001 \
  -H "Content-Type: application/json"
```

### Get Compliance Status
```bash
curl -X GET http://localhost:3000/compliance/status/ASSET001 \
  -H "Content-Type: application/json"
```

## Testing Workflow

### Complete Workflow Example

1. **Create an actor (farmer)**
```bash
curl -X POST http://localhost:3000/actors/create \
  -H "Content-Type: application/json" \
  -d '{
    "actorId": "FARMER001",
    "actorType": "FARMER",
    "name": "Green Valley Farm",
    "registryReference": "REG789012",
    "contactDetails": {
      "email": "contact@greenvalley.com",
      "phone": "+1234567890"
    },
    "certificationIds": []
  }'
```

2. **Declare an asset (wheat)**
```bash
curl -X POST http://localhost:3000/assets/declare \
  -H "Content-Type: application/json" \
  -d '{
    "assetId": "WHEAT001",
    "assetType": "ORGANIC_WHEAT",
    "commodityCode": "WHT001",
    "variety": "Organic Hard Red",
    "quantity": 2000,
    "unit": "kg",
    "originActorId": "FARMER001",
    "originLocation": {
      "latitude": 40.7128,
      "longitude": -74.0060
    },
    "harvestDate": "2024-03-20"
  }'
```

3. **Create a facility**
```bash
curl -X POST http://localhost:3000/facilities/create \
  -H "Content-Type: application/json" \
  -d '{
    "facilityId": "WAREHOUSE001",
    "facilityType": "WAREHOUSE",
    "ownerActorId": "FARMER001",
    "location": {
      "latitude": 40.7128,
      "longitude": -74.0060
    },
    "registrationNumber": "WH_REG001",
    "capacity": 100000
  }'
```

4. **Create a dispatch event**
```bash
curl -X POST http://localhost:3000/events/create \
  -H "Content-Type: application/json" \
  -d '{
    "eventId": "DISPATCH001",
    "eventType": "Dispatch",
    "assetId": "WHEAT001",
    "actorFrom": "FARMER001",
    "actorTo": "PROCESSOR001",
    "facilityId": "WAREHOUSE001",
    "quantity": 1500,
    "unit": "kg"
  }'
```

5. **Get compliance status**
```bash
curl -X GET http://localhost:3000/compliance/status/WHEAT001 \
  -H "Content-Type: application/json"
```

6. **Get full traceability report**
```bash
curl -X GET http://localhost:3000/compliance/report/WHEAT001 \
  -H "Content-Type: application/json"
```

## Error Handling

All APIs return appropriate HTTP status codes:
- `200` - Success
- `400` - Bad Request (missing required fields)
- `404` - Not Found (resource doesn't exist)
- `500` - Internal Server Error (chaincode or network issues)

Error responses include:
```json
{
  "error": "Error description",
  "message": "Detailed error message"
}
```
