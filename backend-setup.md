# OATS Traceability Backend Service

A comprehensive Node.js backend service for the OATS (Organic Agricultural Traceability System) built with Express.js and Hyperledger Fabric SDK.

## Features

- **Actor Management**: Create and manage farmers, processors, exporters
- **Asset Management**: Declare and track agricultural products
- **Facility Management**: Manage processing facilities
- **Traceability Events**: Track all supply chain events
- **Transformations**: Track processing stages
- **Compliance Documents**: Manage certificates and audits
- **Advanced Queries**: Asset lineage, history, compliance reports

## Installation

1. Install dependencies:
```bash
npm install
```

2. Set up Fabric wallet and enroll user:
```bash
# Run from the oats-network directory
cd ../
# Enroll admin and register user (if not already done)
./organizations/fabric-ca/registerEnroll.sh
cd backend
```

3. Start the server:
```bash
npm start
```

For development with auto-reload:
```bash
npm run dev
```

## API Endpoints

### Actors
- `POST /api/actors/create` - Create a new actor
- `GET /api/actors/:actorId` - Get actor details
- `PUT /api/actors/:actorId/status` - Update actor status

### Assets
- `POST /api/assets/declare` - Declare a new asset
- `GET /api/assets/:assetId` - Get asset details
- `PUT /api/assets/:assetId/quantity` - Update asset quantity
- `GET /api/assets/:assetId/history` - Get asset history
- `GET /api/assets/:assetId/lineage` - Get asset lineage
- `GET /api/assets/actor/:actorId` - Query assets by actor
- `GET /api/assets/commodity/:commodityCode` - Query assets by commodity

### Facilities
- `POST /api/facilities/create` - Create a new facility
- `GET /api/facilities/:facilityId` - Get facility details

### Events
- `POST /api/events/create` - Create a traceability event
- `GET /api/events/:eventId` - Get event details
- `GET /api/events/asset/:assetId` - Query events by asset
- `GET /api/events/type/:eventType` - Query events by type

### Transformations
- `POST /api/transformations/create` - Create a transformation
- `GET /api/transformations/:processId` - Get transformation details

### Documents
- `POST /api/documents/create` - Create a compliance document
- `GET /api/documents/:documentId` - Get document details

### Reports
- `GET /api/reports/traceability/:assetId` - Get full traceability report
- `GET /api/reports/compliance/:assetId` - Get compliance status

## Environment Variables

Create a `.env` file with the following variables:

```env
FABRIC_NETWORK_NAME=oatschannel
FABRIC_CONTRACT_NAME=oats-traceability
ORG_NAME=TRST01
ORG_MSP=TRST01MSP
PEER_NAME=peer0.trst01.example.com
PEER_HOST=localhost:7061
ORDERER_NAME=orderer1.example.com
ORDERER_HOST=localhost:7060
CRYPTO_PATH=../organizations/peerOrganizations/trst01.example.com
CERT_PATH=../organizations/peerOrganizations/trst01.example.com/users/Admin@trst01.example.com/msp
TLS_CERT_PATH=../organizations/peerOrganizations/trst01.example.com/peers/peer0.trst01.example.com/tls/ca.crt
CONNECTION_PROFILE_PATH=../organizations/peerOrganizations/trst01.example.com/connection-trst01.json
PORT=3000
NODE_ENV=development
```

## API Documentation

Visit `http://localhost:3000/api` for complete API documentation.

## Health Check

Visit `http://localhost:3000/health` to check service status.

## Error Handling

All endpoints return consistent JSON responses:

```json
{
  "success": true,
  "message": "Operation successful",
  "data": { ... }
}
```

Error responses:
```json
{
  "success": false,
  "message": "Error description",
  "error": "Detailed error message"
}
```

## License

Apache-2.0
