# OATS Network - Single Organization (TRST01) with 3 RAFT Orderers

This folder is a customized copy of Fabric sample `test-network`, adapted to this target topology:

- **TRST01 org**: `peer0`, `peer1` (optional backup)
- **RAFT orderers**: `orderer1`, `orderer2`, `orderer3`
- **Channel**: `oatschannel`

All commands below assume you are in:

```bash
cd ~/fabric-dev/fabric-samples/oats-network
```

---

## Architecture Overview

```
TRST01 Organization
├─ peer0.trst01.example.com:7061
└─ peer1.trst01.example.com:8061 (backup)

RAFT Orderers
├─ orderer1.example.com:7060 (admin: 7063, metrics: 9463)
├─ orderer2.example.com:8060 (admin: 8063, metrics: 9466)
└─ orderer3.example.com:9060 (admin: 9063, metrics: 9467)
```

**Purpose**: Immutable ledger operated by TRST01 organization with 3-node RAFT consensus for high availability.

---

## Phase 1 — Verify environment on official `test-network`

Before customizations, verify your Fabric binaries, Docker, and sample scripts work.

```bash
cd ~/fabric-dev/fabric-samples/test-network
./network.sh up
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
./network.sh down
```

If this succeeds, the environment is healthy.

---

## Phase 2 — Update crypto-config for TRST01 organization

Modified `organizations/cryptogen/crypto-config-org1.yaml`:

- Changed `Org1` to `TRST01`
- Domain: `trst01.example.com`
- Peer count: 2 (peer0, peer1)

```yaml
- Name: TRST01
  Domain: trst01.example.com
  Template:
    Count: 2
```

---

## Phase 3 — Configure 3 RAFT orderers

Modified `organizations/cryptogen/crypto-config-orderer.yaml`:

```yaml
Specs:
  - Hostname: orderer1
    SANS:
      - localhost
  - Hostname: orderer2
    SANS:
      - localhost
  - Hostname: orderer3
    SANS:
      - localhost
```

---

## Phase 4 — Update configtx.yaml for TRST01 and RAFT consenters

Modified `configtx/configtx.yaml`:

- Defined `TRST01MSP` organization
- Set 3 RAFT consenters:
  - `orderer1.example.com:7050`
  - `orderer2.example.com:8050`
  - `orderer3.example.com:9050`
- Updated application organizations to include only `TRST01`

---

## Phase 5 — Update Docker compose files

Updated port mappings to avoid conflicts with other networks:

### compose-test-net.yaml
- **orderer1**: 7060 (orderer), 7063 (admin), 9463 (metrics)
- **orderer2**: 8060 (orderer), 8063 (admin), 9466 (metrics)
- **orderer3**: 9060 (orderer), 9063 (admin), 9467 (metrics)
- **peer0**: 7061 (peer), 9468 (metrics)
- **peer1**: 8061 (peer), 9469 (metrics)

### docker-compose-test-net.yaml
- Updated service definitions for TRST01 peers
- Set DOCKER_SOCK environment variable

---

## Phase 6 — Generate crypto material

```bash
export PATH=$PWD/../bin:$PATH

# Optional cleanup (fresh crypto)
rm -rf organizations/peerOrganizations organizations/ordererOrganizations

# Generate TRST01 organization crypto (2 peers)
cryptogen generate --config=organizations/cryptogen/crypto-config-org1.yaml --output=organizations

# Generate orderer crypto (3 orderers)
cryptogen generate --config=organizations/cryptogen/crypto-config-orderer.yaml --output=organizations
```

---

## Phase 7 — Generate channel block for `oatschannel`

```bash
export FABRIC_CFG_PATH=$PWD/configtx
mkdir -p channel-artifacts

configtxgen -profile ChannelUsingRaft \
  -outputBlock channel-artifacts/oatschannel.block \
  -channelID oatschannel

# Reset FABRIC_CFG_PATH for peer operations
unset FABRIC_CFG_PATH
export FABRIC_CFG_PATH=$PWD/compose/docker/peercfg
```

---

## Phase 8 — Clean up previous networks (if needed)

```bash
# Clean up orphan containers and volumes
docker-compose -f compose/compose-test-net.yaml -f compose/docker/docker-compose-test-net.yaml down --remove-orphans -v

# Set DOCKER_SOCK variable
export DOCKER_SOCK=/var/run/docker.sock
```

---

## Phase 9 — Start the network (3 orderers + 2 peers)

```bash
# Start all containers
docker-compose -f compose/compose-test-net.yaml -f compose/docker/docker-compose-test-net.yaml up -d

# Check they are up
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

You should see:
- **Orderers**: `orderer1.example.com`, `orderer2.example.com`, `orderer3.example.com`
- **Peers**: `peer0.trst01.example.com`, `peer1.trst01.example.com`

---

## Phase 10 — Join orderers to the channel

Join `oatschannel` on each orderer using the admin endpoint.

```bash
# Join orderer1
osnadmin channel join \
  --channelID oatschannel \
  --config-block channel-artifacts/oatschannel.block \
  -o localhost:7063 \
  --ca-file organizations/ordererOrganizations/example.com/orderers/orderer1.example.com/tls/ca.crt \
  --client-cert organizations/ordererOrganizations/example.com/orderers/orderer1.example.com/tls/server.crt \
  --client-key organizations/ordererOrganizations/example.com/orderers/orderer1.example.com/tls/server.key

# Join orderer2
osnadmin channel join \
  --channelID oatschannel \
  --config-block channel-artifacts/oatschannel.block \
  -o localhost:8063 \
  --ca-file organizations/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/ca.crt \
  --client-cert organizations/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/server.crt \
  --client-key organizations/ordererOrganizations/example.com/orderers/orderer2.example.com/tls/server.key

# Join orderer3
osnadmin channel join \
  --channelID oatschannel \
  --config-block channel-artifacts/oatschannel.block \
  -o localhost:9063 \
  --ca-file organizations/ordererOrganizations/example.com/orderers/orderer3.example.com/tls/ca.crt \
  --client-cert organizations/ordererOrganizations/example.com/orderers/orderer3.example.com/tls/server.crt \
  --client-key organizations/ordererOrganizations/example.com/orderers/orderer3.example.com/tls/server.key
```

Verify each orderer sees the channel:

```bash
osnadmin channel list -o localhost:7063 \
  --ca-file organizations/ordererOrganizations/example.com/orderers/orderer1.example.com/tls/ca.crt \
  --client-cert organizations/ordererOrganizations/example.com/orderers/orderer1.example.com/tls/server.crt \
  --client-key organizations/ordererOrganizations/example.com/orderers/orderer1.example.com/tls/server.key
```

---

## Phase 11 — Join peers to the channel

Important: the `peer` CLI needs `core.yaml`. For this repo, use:

```bash
export CORE_PEER_TLS_ENABLED=true
```

### TRST01 Peers

```bash
# peer0.trst01 (7061)
export CORE_PEER_LOCALMSPID=TRST01MSP
export CORE_PEER_MSPCONFIGPATH=$PWD/organizations/peerOrganizations/trst01.example.com/users/Admin@trst01.example.com/msp
export CORE_PEER_ADDRESS=localhost:7061
export CORE_PEER_TLS_ROOTCERT_FILE=$PWD/organizations/peerOrganizations/trst01.example.com/peers/peer0.trst01.example.com/tls/ca.crt
peer channel join -b channel-artifacts/oatschannel.block

# peer1.trst01 (8061)
export CORE_PEER_LOCALMSPID=TRST01MSP
export CORE_PEER_MSPCONFIGPATH=$PWD/organizations/peerOrganizations/trst01.example.com/users/Admin@trst01.example.com/msp
export CORE_PEER_ADDRESS=localhost:8061
export CORE_PEER_TLS_ROOTCERT_FILE=$PWD/organizations/peerOrganizations/trst01.example.com/peers/peer1.trst01.example.com/tls/ca.crt
peer channel join -b channel-artifacts/oatschannel.block
```

Quick verify (run per peer after exporting env for that peer):

```bash
peer channel list
```

---

## What was changed in files (summary)

### Crypto (cryptogen)
- `organizations/cryptogen/crypto-config-org1.yaml` → **TRST01**, `Count: 2`, `Domain: trst01.example.com`
- `organizations/cryptogen/crypto-config-orderer.yaml` → **orderer1/2/3** (removed extra orderers)

### Channel config (configtx)
- `configtx/configtx.yaml`
  - Defines `TRST01MSP` organization
  - Sets **Anchor Peer**: `peer0.trst01.example.com:7051`
  - Sets **etcdraft Consenters**:
    - `orderer1.example.com:7050`
    - `orderer2.example.com:8050`
    - `orderer3.example.com:9050`

### Docker compose
Updated to run 3 orderers + 2 peers with unique port mappings:
- `compose/compose-test-net.yaml`
- `compose/docker/docker-compose-test-net.yaml`

---

## How to inspect the running network

### What containers are running / since when
```bash
docker ps --format "table {{.Names}}\t{{.RunningFor}}\t{{.Status}}\t{{.Ports}}"
```

### What channels each orderer has
```bash
osnadmin channel list -o localhost:7063 \
  --ca-file organizations/ordererOrganizations/example.com/orderers/orderer1.example.com/tls/ca.crt \
  --client-cert organizations/ordererOrganizations/example.com/orderers/orderer1.example.com/tls/server.crt \
  --client-key organizations/ordererOrganizations/example.com/orderers/orderer1.example.com/tls/server.key
```

### What channel(s) a peer has joined
(Set the peer env like in the join section, then:)

```bash
peer channel list
```

---

## Stop / start tomorrow (two common options)

### Option A — Stop and resume later (keep all ledgers/channel membership)
This is the usual "pause and continue tomorrow".

Stop containers **without deleting volumes**:

```bash
docker-compose -f compose/compose-test-net.yaml -f compose/docker/docker-compose-test-net.yaml stop
```

Start them again:

```bash
docker-compose -f compose/compose-test-net.yaml -f compose/docker/docker-compose-test-net.yaml start
```

In this option:
- You **do not** need to re-run `cryptogen`, `configtxgen`, `osnadmin join`, or `peer channel join`.
- The channel and ledgers are preserved in Docker volumes.

### Option B — Tear down and recreate from scratch (clean reset)
Use this if you want a clean environment (it deletes volumes/ledgers).

```bash
docker-compose -f compose/compose-test-net.yaml -f compose/docker/docker-compose-test-net.yaml down -v
rm -rf channel-artifacts system-genesis-block
```

Then redo:
- Phase 6 (crypto) if you also removed `organizations/...`
- Phase 7 (regenerate `oatschannel.block`)
- Phase 9 (up)
- Phase 10 (orderer joins + peer joins)

---

## Troubleshooting notes (common gotchas)

### `peer` CLI: "core config file not found"
If you see:

> `Config File "core" Not Found ...`

you likely still have:

```bash
export FABRIC_CFG_PATH=$PWD/configtx
```

Fix by using:

```bash
unset FABRIC_CFG_PATH
export FABRIC_CFG_PATH=$PWD/compose/docker/peercfg
```

### Orderers exiting / TLS handshake errors / `system-channel` replication panic
If orderers exit and logs mention `system-channel` replication, it's usually old ledger state.

Fix with a clean orderer reset:

```bash
docker-compose -f compose/compose-test-net.yaml -f compose/docker/docker-compose-test-net.yaml down -v
docker-compose -f compose/compose-test-net.yaml -f compose/docker/docker-compose-test-net.yaml up -d
```

### Port conflicts with other networks
If you get "port is already allocated" errors, the port mappings in this README are designed to avoid conflicts with standard Fabric test networks. If you still have conflicts, you can modify the ports in `compose/compose-test-net.yaml`.

---

## Network Status

✅ **Completed**: Single organization TRST01 with 3 RAFT orderers for immutable ledger operations
✅ **High Availability**: 3-node RAFT consensus ensures no single point of failure
✅ **Scalability**: Optional peer1 provides backup capability
✅ **Ready for Chaincode**: Network is ready for smart contract deployment and transaction processing

Your OATS network is now ready for creating an immutable ledger!

Note that running the createChannel command will start the network, if it is not already running.

Before you can deploy the test network, you need to follow the instructions to [Install the Samples, Binaries and Docker Images](https://hyperledger-fabric.readthedocs.io/en/latest/install.html) in the Hyperledger Fabric documentation.

## Using the Peer commands

The `setOrgEnv.sh` script can be used to set up the environment variables for the organizations, this will help to be able to use the `peer` commands directly.

First, ensure that the peer binaries are on your path, and the Fabric Config path is set assuming that you're in the `test-network` directory.

```bash
 export PATH=$PATH:$(realpath ../bin)
 export FABRIC_CFG_PATH=$(realpath ../config)
```

You can then set up the environment variables for each organization. The `./setOrgEnv.sh` command is designed to be run as follows.

```bash
export $(./setOrgEnv.sh Org2 | xargs)
```

(Note bash v4 is required for the scripts.)

You will now be able to run the `peer` commands in the context of Org2. If a different command prompt, you can run the same command with Org1 instead.
The `setOrgEnv` script outputs a series of `<name>=<value>` strings. These can then be fed into the export command for your current shell.

## Chaincode-as-a-service

To learn more about how to use the improvements to the Chaincode-as-a-service please see this [tutorial](./test-network/../CHAINCODE_AS_A_SERVICE_TUTORIAL.md). It is expected that this will move to augment the tutorial in the [Hyperledger Fabric ReadTheDocs](https://hyperledger-fabric.readthedocs.io/en/release-2.4/cc_service.html)


## Podman

*Note - podman support should be considered experimental but the following has been reported to work with podman 4.1.1 on Mac. If you wish to use podman a LinuxVM is recommended.*

Fabric's `install-fabric.sh` script has been enhanced to support using `podman` to pull down images and tag them rather than docker. The images are the same, just pulled differently. Simply specify the 'podman' argument when running the `install-fabric.sh` script. 

Similarly, the `network.sh` script has been enhanced so that it can use `podman` and `podman-compose` instead of docker. Just set the environment variable `CONTAINER_CLI` to `podman` before running the `network.sh` script:

```bash
CONTAINER_CLI=podman ./network.sh up
````

As there is no Docker-Daemon when using podman, only the `./network.sh deployCCAAS` command will work. Following the Chaincode-as-a-service Tutorial above should work. 


