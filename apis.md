# OATS Traceability API Testing with cURL

This document contains all cURL commands to test the OATS Traceability Backend Service.

## Server Setup

Make sure the backend server is running:
```bash
cd backend
npm install
npm start
```

Server runs on: `http://localhost:3000`

---

## Health Check

```bash
curl -X GET http://localhost:3000/health
```

---

## Actor Management APIs

### 1. Create Farmer Actor

```bash
curl -X POST http://localhost:3000/api/actors/create \
  -H "Content-Type: application/json" \
  -d '{
    "actorId": "ACT-FARMER-001",
    "actorType": "Farmer",
    "name": "Green Valley Farms",
    "registryReference": "FARM-REG-001",
    "contactDetails": {
      "phone": "+91-9876543210",
      "email": "farmer@greenvalley.com"
    },
    "certificationIds": ["CERT-ORG-001", "CERT-FAIR-001"]
  }'
```

### 2. Create Processor Actor

```bash
curl -X POST http://localhost:3000/api/actors/create \
  -H "Content-Type: application/json" \
  -d '{
    "actorId": "ACT-PROCESSOR-001",
    "actorType": "Processor",
    "name": "Agri Processing Ltd",
    "registryReference": "PROC-REG-001",
    "contactDetails": {
      "phone": "+91-9876543211",
      "email": "info@agriprocess.com"
    },
    "certificationIds": ["CERT-ISO-001", "CERT-HACCP-001"]
  }'
```

### 3. Get Actor Details

```bash
curl -X GET http://localhost:3000/api/actors/ACT-FARMER-001
```

### 4. Update Actor Status

```bash
curl -X PUT http://localhost:3000/api/actors/ACT-FARMER-001/status \
  -H "Content-Type: application/json" \
  -d '{
    "newStatus": "Active"
  }'
```

---

## Facility Management APIs

### 5. Create Facility

```bash
curl -X POST http://localhost:3000/api/facilities/create \
  -H "Content-Type: application/json" \
  -d '{
    "facilityId": "FAC-PROC-001",
    "facilityType": "Processor",
    "ownerActorId": "ACT-PROCESSOR-001",
    "location": {
      "latitude": 19.7515,
      "longitude": 75.7139,
      "district": "Nashik",
      "state": "Maharashtra",
      "country": "India"
    },
    "registrationNumber": "REG-FAC-001",
    "capacity": 50000,
    "complianceCertificates": ["CERT-ISO-001", "CERT-HACCP-001"]
  }'
```

### 6. Get Facility Details

```bash
curl -X GET http://localhost:3000/api/facilities/FAC-PROC-001
```

---

## Asset Management APIs

### 7. Declare Asset (Organic Cotton)

```bash
curl -X POST http://localhost:3000/api/assets/declare \
  -H "Content-Type: application/json" \
  -d '{
    "assetId": "AST-COTTON-001",
    "assetType": "Raw",
    "commodityCode": "COTTON-ORG",
    "variety": "Organic Long Staple",
    "quantity": 1000,
    "unit": "kg",
    "originActorId": "ACT-FARMER-001",
    "originLocation": {
      "latitude": 19.7515,
      "longitude": 75.7139,
      "district": "Nashik",
      "state": "Maharashtra",
      "country": "India"
    },
    "harvestDate": "2024-03-15",
    "certificationReference": ["CERT-ORG-001", "CERT-FAIR-001"]
  }'
```

### 8. Get Asset Details

```bash
curl -X GET http://localhost:3000/api/assets/AST-COTTON-001
```

### 9. Update Asset Quantity

```bash
curl -X PUT http://localhost:3000/api/assets/AST-COTTON-001/quantity \
  -H "Content-Type: application/json" \
  -d '{
    "newQuantity": 950,
    "reason": "Quality inspection deduction"
  }'
```

### 10. Get Asset History

```bash
curl -X GET http://localhost:3000/api/assets/AST-COTTON-001/history
```

### 11. Get Asset Lineage

```bash
curl -X GET http://localhost:3000/api/assets/AST-COTTON-001/lineage
```

### 12. Query Assets by Actor

```bash
curl -X GET http://localhost:3000/api/assets/actor/ACT-FARMER-001
```

### 13. Query Assets by Commodity

```bash
curl -X GET http://localhost:3000/api/assets/commodity/COTTON-ORG
```

---

## Traceability Event APIs

### 14. Create Declare Event

```bash
curl -X POST http://localhost:3000/api/events/create \
  -H "Content-Type: application/json" \
  -d '{
    "eventId": "EVT-DEC-001",
    "eventType": "Declare",
    "assetId": "AST-COTTON-001",
    "actorFrom": "ACT-FARMER-001",
    "actorTo": null,
    "facilityId": null,
    "quantity": 1000,
    "unit": "kg",
    "eventMetadata": {
      "season": "Kharif",
      "grade": "A"
    }
  }'
```

### 15. Create Request Event

```bash
curl -X POST http://localhost:3000/api/events/create \
  -H "Content-Type: application/json" \
  -d '{
    "eventId": "EVT-REQ-001",
    "eventType": "Request",
    "assetId": "AST-COTTON-001",
    "actorFrom": "ACT-PROCESSOR-001",
    "actorTo": "ACT-FARMER-001",
    "facilityId": null,
    "quantity": 1000,
    "unit": "kg",
    "eventMetadata": {
      "price_per_kg": 150,
      "delivery_date": "2024-03-20"
    }
  }'
```

### 16. Create Approve Event

```bash
curl -X POST http://localhost:3000/api/events/create \
  -H "Content-Type: application/json" \
  -d '{
    "eventId": "EVT-APP-001",
    "eventType": "Approve",
    "assetId": "AST-COTTON-001",
    "actorFrom": "ACT-FARMER-001",
    "actorTo": "ACT-PROCESSOR-001",
    "facilityId": null,
    "quantity": 1000,
    "unit": "kg",
    "eventMetadata": {
      "approval_ref": "APR-2024-001"
    }
  }'
```

### 17. Create Dispatch Event

```bash
curl -X POST http://localhost:3000/api/events/create \
  -H "Content-Type: application/json" \
  -d '{
    "eventId": "EVT-DIS-001",
    "eventType": "Dispatch",
    "assetId": "AST-COTTON-001",
    "actorFrom": "ACT-FARMER-001",
    "actorTo": "ACT-PROCESSOR-001",
    "facilityId": "FAC-PROC-001",
    "quantity": 1000,
    "unit": "kg",
    "eventMetadata": {
      "transport_type": "Truck",
      "vehicle_number": "MH-12-AB-1234"
    }
  }'
```

### 18. Create Receipt Event

```bash
curl -X POST http://localhost:3000/api/events/create \
  -H "Content-Type: application/json" \
  -d '{
    "eventId": "EVT-REC-001",
    "eventType": "Receipt",
    "assetId": "AST-COTTON-001",
    "actorFrom": "ACT-FARMER-001",
    "actorTo": "ACT-PROCESSOR-001",
    "facilityId": "FAC-PROC-001",
    "quantity": 980,
    "unit": "kg",
    "eventMetadata": {
      "received_quantity": 980,
      "shortage_reason": "Transit loss"
    }
  }'
```

### 19. Get Event Details

```bash
curl -X GET http://localhost:3000/api/events/EVT-DEC-001
```

### 20. Query Events by Asset

```bash
curl -X GET http://localhost:3000/api/events/asset/AST-COTTON-001
```

### 21. Query Events by Type

```bash
curl -X GET http://localhost:3000/api/events/type/Declare
```

---

## Transformation APIs

### 22. Create Transformation (Ginning)

```bash
curl -X POST http://localhost:3000/api/transformations/create \
  -H "Content-Type: application/json" \
  -d '{
    "processId": "TRANS-GIN-001",
    "processType": "Ginning",
    "inputAssets": ["AST-COTTON-001"],
    "outputAssets": ["AST-COTTON-GINNED-001"],
    "facilityId": "FAC-PROC-001",
    "yieldRatio": 0.85
  }'
```

### 23. Get Transformation Details

```bash
curl -X GET http://localhost:3000/api/transformations/TRANS-GIN-001
```

---

## Compliance Document APIs

### 24. Create Compliance Document

```bash
curl -X POST http://localhost:3000/api/documents/create \
  -H "Content-Type: application/json" \
  -d '{
    "documentId": "DOC-ORG-001",
    "documentType": "Certificate",
    "issuingAuthority": "Organic Certification Board",
    "relatedAsset": "AST-COTTON-001",
    "validFrom": "2024-01-01",
    "validTo": "2025-12-31",
    "digitalHash": "hash_abc123def456",
    "storageUri": "ipfs://QmXxx..."
  }'
```

### 25. Get Document Details

```bash
curl -X GET http://localhost:3000/api/documents/DOC-ORG-001
```

---

## Report APIs

### 26. Get Full Traceability Report

```bash
curl -X GET http://localhost:3000/api/reports/traceability/AST-COTTON-001
```

### 27. Get Compliance Status

```bash
curl -X GET http://localhost:3000/api/reports/compliance/AST-COTTON-001
```

---

## API Documentation

### 28. Get API Documentation

```bash
curl -X GET http://localhost:3000/api
```

---

## Testing Complete Supply Chain

Run these commands in sequence to test a complete agricultural supply chain:

1. **Setup**: Commands 1-6 (Create actors and facilities)
2. **Asset Declaration**: Commands 7-8 (Declare cotton asset)
3. **Supply Chain Events**: Commands 14-18 (Declare → Request → Approve → Dispatch → Receipt)
4. **Processing**: Commands 22-23 (Ginning transformation)
5. **Compliance**: Commands 24-25 (Add compliance document)
6. **Reporting**: Commands 26-27 (Generate reports)

---

## Expected Response Format

All APIs return consistent JSON responses:

**Success Response:**
```json
{
  "success": true,
  "message": "Operation successful",
  "data": { ... }
}
```

**Error Response:**
```json
{
  "success": false,
  "message": "Error description",
  "error": "Detailed error message"
}
```

---

## Troubleshooting

1. **Connection Issues**: Ensure Fabric network is running and backend server is started
2. **Wallet Issues**: Make sure the Fabric wallet is set up and user is enrolled
3. **Certificate Issues**: Verify certificate paths in `.env` file
4. **Port Conflicts**: Ensure port 3000 is not in use

For detailed logs, check the backend server console output.
