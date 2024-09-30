# Linera setup node

---

## Running a Validator Node with Docker Compose

### One-Click Deploy

**Note**: This section is tested under Linux.

To deploy a Linera validator, follow these steps after downloading the [Linera Protocol](https://github.com/linera-io/linera-protocol) repository and checking out the `testnet_boole` branch:

1. Fetch the latest changes and check out the branch:
   ```bash
   git fetch origin
   git checkout -t origin/testnet_boole
   ```

2. Deploy the validator using the script:
   ```bash
   ./scripts/deploy-validator.sh <hostname>
   ```

For example:
```bash
./scripts/deploy-validator.sh linera.mydomain.com
```

After the script runs, your public key will be printed:
```bash
Public Key: 92f934525762a9ed99fcc3e3d3e35a825235dae133f2682b78fe22a742bac196
```

Provide the public key and the hostname to the Linera Protocol core team to join the next epoch.

### Manual Installation

This section explains how to manually install and run a Linera validator node using Docker Compose.

#### Docker Compose Requirements

To install Docker Compose, refer to the official [Docker docs](https://docs.docker.com/compose/install/).

#### Installing the Linera Toolchain

Check out the `testnet_boole` branch when installing the toolchain. Follow the instructions in the [Linera Toolchain installation guide](https://github.com/linera-io/linera-protocol#installation).

#### Validator Configuration

1. **Infrastructure Configuration**:  
   Validators running on Docker Compose don't come with a load balancer for TLS termination. Ensure your load balancer supports:
   - HTTP/2 and gRPC connections
   - Long-lived HTTP/2 connections
   - A maximum body size of 20 MB
   - TLS termination with a certificate signed by a known CA
   - Traffic redirection from port `443` to `19100`

2. **Creating Your Validator Configuration**:  
   Validators are configured with a TOML file. Use the following template:
   ```toml
   server_config_path = "server.json"
   host = "<your-host>"
   port = 19100
   metrics_host = "proxy"
   metrics_port = 21100
   internal_host = "proxy"
   internal_port = 20100
   [external_protocol]
   Grpc = "ClearText"
   [internal_protocol]
   Grpc = "ClearText"
   [[shards]]
   host = "shard"
   port = 19100
   metrics_host = "shard"
   metrics_port = 21100
   ```

3. **Genesis Configuration**:  
   The genesis configuration describes the initial validator set and can be found in a public bucket:
   ```bash
   wget "https://storage.googleapis.com/linera-io-dev-public/testnet-boole/genesis.json"
   ```

4. **Creating Your Keys**:  
   Generate validator private keys with the `linera-server` binary:
   ```bash
   linera-server generate --validators /path/to/validator/configuration.toml
   ```

   Example output:
   ```bash
   Public Key: 92f934525762a9ed99fcc3e3d3e35a825235dae133f2682b78fe22a742bac196
   ```

### Building the Docker Image

To build the Linera Docker image, run:
```bash
docker build -f docker/Dockerfile . -t linera
```

### Running a Validator Node

With the genesis configuration and server configuration in place, start the validator:
```bash
cd docker && docker compose up -d
```

The deployment will run in detached mode. ScyllaDB and other dependencies will be downloaded and started automatically.

---

