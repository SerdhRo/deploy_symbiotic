## Sumbiotic installation

To set up a Symbiotic validator node, you'll need to follow a series of steps involving configuring your environment, building the node, and interacting with smart contracts. Here's a comprehensive guide to get you started, focusing on Symbiotic's Cosmos SDK-based setup.

### Prerequisites
Ensure you have the following installed:
- **Go** (version 1.20 or higher)
- **Git** and **Make**
- Access to Ethereum's RPC and a beacon chain client for the Holešky testnet

### Step 1: Environment Setup
First, set up environment variables for RPC URLs:
```bash
export BEACON_API_URLS=<your_beacon_rpc_url>
export ETH_API_URLS=<your_eth_rpc_url>
```
Symbiotic's contracts are deployed on the Holešky testnet, so these URLs should be testnet-specific. It’s recommended to run your own beacon client for security.

### Step 2: Cloning and Building the Node
Clone the Symbiotic repository and build the node:
```bash
git clone https://github.com/symbioticfi/cosmos-sdk
cd cosmos-sdk
make build-sym
```
The built binary (`symd`) will be located in the `build` directory. Add it to your `$PATH` for easy access.

### Step 3: Initialize the Node
Initialize your node with the chain ID:
```bash
symd init <your_moniker> --chain-id chain-D6yfsZ
```
Copy the `genesis.json` file from the `stubchain` directory:
```bash
cp stubchain/genesis.json ~/.symapp/config/
```

### Step 4: Configuring Peers
Edit the `config.toml` file to add the necessary peers:
```bash
# Example configuration in ~/.symapp/config/config.toml
[p2p]
seeds = "716f3d6c6b6cdb2c3e980641c8a2f2e2a5089cbb@node-01.stubchain.symbiotic.fi:26656,30a07da2029074bd48d5bf53423f8f4ebe706167@node-04.stubchain.symbiotic.fi:26656"
persistent_peers = "f3e411655b1af5f83f31ecd8acc2567f358ac18f@node-06.stubchain.symbiotic.fi:26656"
```

### Step 5: Creating and Managing Operator Keys
Generate operator keys using the command:
```bash
symd keys add <key_name> --keyring-backend <keyring_backend>
```

### Step 6: Running the Node
Start the node with:
```bash
symd start
```
Monitor the sync status:
```bash
curl -s 127.0.0.1:26657/status | jq '.result.sync_info.catching_up'
```
The status should return `false` once the node is fully synchronized.

### Step 7: Validator Creation
After synchronization and operator registration, you can create a validator:
1. Create a `validator.json` file with your details:
    ```json
    {
      "pubkey": {
        "@type": "/cosmos.crypto.ed25519.PubKey",
        "key": "<your pub_key from priv_validator_key.json>"
      },
      "moniker": "<your node moniker>",
      "commission-rate": "0.1",
      "commission-max-rate": "0.1",
      "commission-max-change-rate": "0.1"
    }
    ```
2. Submit the transaction:
    ```bash
    symd tx symStaking create-validator validator.json \
    --chain-id=chain-D6yfsZ \
    --from=<key_name> \
    --keyring-backend=<keyring_backend>
    ```
Verify your validator’s status:
```bash
symd query symStaking validators
```

### Step 8: Monitoring and Maintenance
Regularly monitor your node’s performance and set up alerting systems. For critical updates or node crashes, configure automatic restarts to minimize downtime.

### Additional Notes
- Symbiotic utilizes middleware contracts and components like `OperatorRegistry` for operator management and network integration.
- Validators are required to register with both Ethereum contracts and Symbiotic’s network-specific components to complete the setup.
