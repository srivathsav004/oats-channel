# OATS Traceability Backend Service

A comprehensive Node.js backend service for the OATS (Organic Agricultural Traceability System) built with Express.js and Hyperledger Fabric CLI.

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

2. Start the server:
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
CHANNEL_NAME=oatschannel
CC_NAME=oats-traceability
ORDERER_ADDRESS=localhost:7060
ORDERER_TLS_HOST=orderer1.example.com
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

## Architecture

This backend uses the Hyperledger Fabric CLI directly instead of the SDK, making it simpler and more reliable. It executes `peer` commands using Node.js `execFile` and parses the JSON responses.

## License

Apache-2.0
