# OATS Traceability Backend Service

This backend service provides REST APIs for interacting with the OATS Traceability Hyperledger Fabric chaincode.

## Setup

1. Install dependencies:
```bash
npm install
```

2. Configure environment variables in `.env`:
```
CHANNEL_NAME=oatschannel
CC_NAME=oats-traceability-javascript
ORDERER_ADDRESS=localhost:7050
CORE_PEER_LOCALMSPID=TRST01MSP
CORE_PEER_ADDRESS=localhost:7051
PORT=3000
```

3. Start the server:
```bash
npm start
```

The server will start on `http://localhost:3000`

## API Endpoints

### Health Check
- `GET /health` - Health check

### Actor Management
- `POST /actors/create` - Create a new actor
- `GET /actors/:actorId` - Get actor details
- `PUT /actors/:actorId/status` - Update actor status

### Asset Management
- `POST /assets/declare` - Declare a new asset
- `GET /assets/:assetId` - Get asset details
- `PUT /assets/:assetId/quantity` - Update asset quantity

### Facility Management
- `POST /facilities/create` - Create a new facility
- `GET /facilities/:facilityId` - Get facility details

### Traceability Events
- `POST /events/create` - Create a traceability event
- `GET /events/:eventId` - Get event details

### Transformations
- `POST /transformations/create` - Create a transformation
- `GET /transformations/:processId` - Get transformation details

### Compliance Documents
- `POST /documents/create` - Create a compliance document
- `GET /documents/:documentId` - Get document details

### Query Functions
- `GET /query/asset/:assetId/history` - Get asset history
- `GET /query/asset/:assetId/lineage` - Get asset lineage
- `GET /query/assets/actor/:actorId` - Query assets by actor
- `GET /query/events/asset/:assetId` - Query events by asset
- `GET /query/events/type/:eventType` - Query events by type
- `GET /query/assets/commodity/:commodityCode` - Query assets by commodity

### Compliance and Audit
- `GET /compliance/report/:assetId` - Get full traceability report
- `GET /compliance/status/:assetId` - Get compliance status

See `curl-examples.md` for detailed curl commands to test all APIs.
