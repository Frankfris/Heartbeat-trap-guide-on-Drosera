## Deploying HeartbeatTrap on Drosera

This guide walks you through creating, configuring, and deploying a HeartbeatTrap on Drosera for automated on‑chain monitoring.

## Step 1 – Create Your Working Directory
```bash
mkdir heartbeat-trap && cd heartbeat-trap
forge init
```

## Step 2 – Add Drosera Contracts
```bash
mkdir -p lib
git clone https://github.com/drosera-network/contracts.git lib/contracts
```

## Step 3 – Create the HeartbeatTrap Contract
Create a new file at src/HeartbeatTrap.sol and paste:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {ITrap} from "../lib/contracts/src/interfaces/ITrap.sol";

interface IMockResponse {
    function isActive() external view returns (bool);
}

contract HeartbeatTrap is ITrap {
    address public constant RESPONSE_CONTRACT = "YOUR RESPONSE CONTRACT ADDRESS";
    string constant discordName = "YOUR DISCORD USERNAME";

    function collect() external view override returns (bytes memory) {
        bool active = IMockResponse(RESPONSE_CONTRACT).isActive();
        return abi.encode(active, discordName);
    }

    function shouldRespond(bytes[] calldata data)
        external
        pure
        override
        returns (bool, bytes memory)
    {
        (bool active, string memory name) = abi.decode(data[0], (bool, string));
        if (!active || bytes(name).length == 0) {
            return (false, bytes(""));
        }
        return (true, abi.encode(name));
    }
}
```

## Step 4 – Create the Mock Active Contract
Create a new file at src/MockActiveContract.sol and paste:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract MockActiveContract {
    function isActive() external pure returns (bool) {
        return true;
    }
}
```

## Step 5 – Deploy the MockActiveContract
Run the following command to deploy the MockActiveContract to the Hoodi Ethereum network:

```bash
forge create src/MockActiveContract.sol:MockActiveContract \
  --rpc-url https://ethereum-hoodi-rpc.publicnode.com \
  --private-key YOUR_PRIVATE_KEY
```
Replace YOUR_PRIVATE_KEY with your testnet private key. Copy the deployed address and replace RESPONSE_CONTRACT in HeartbeatTrap.sol.

## Step 6 – Clean and Build the Project
Before deploying, clean and rebuild the project to ensure everything compiles correctly:

```bash
forge clean
forge build
```
## Step 7 – Configure drosera.toml
Create a file named drosera.toml in the project root and paste the following configuration:  

```toml
ethereum_rpc = "https://ethereum-hoodi-rpc.publicnode.com"
drosera_rpc = "https://relay.hoodi.drosera.io"
eth_chain_id = 560048
drosera_address = "0x91cb447bafc6e0ea0f4fe056f5a9b1f14bb06e5d"

[traps.HeartbeatTrap]
path = "out/HeartbeatTrap.sol/HeartbeatTrap.json"
response_contract = "<deployed response contract address>"
response_function = "handleMissedHeartbeat(uint256)"
cooldown_period_blocks = 33
min_number_of_operators = 1
max_number_of_operators = 2
block_sample_size = 10
private_trap = false
whitelist = []
```

## Step 8 – Install Drosera CLI
Run the following commands to download and install Drosera CLI:

```bash
wget https://github.com/drosera-network/releases/releases/download/v1.20.0/drosera-v1.20.0-x86_64-unknown-linux-gnu.tar.gz
tar -xzf drosera-v1.20.0-x86_64-unknown-linux-gnu.tar.gz
chmod +x drosera
mkdir -p ~/.drosera/bin
mv drosera ~/.drosera/bin/
echo 'export PATH=$PATH:~/.drosera/bin' >> ~/.bashrc
source ~/.bashrc
drosera --version
```

## Step 9 – Deploy the Trap
Run the following command to apply your trap configuration to the Drosera network:  

```bash
drosera apply \
  --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com \
  --private-key YOUR_PRIVATE_KEY
```
## ✅ Successful Deployment and Dry Run
After running drosera apply, you should see output similar to:

```bash
Response function verified ✅
Testing trap(s) execution...
Dry run passed ✅
Created Trap Config for HeartbeatTrap
```

## Step 10 – Test the Trap with a Dry Run
Run the following command to simulate trap execution without sending real transactions:  

```bash
drosera dryrun
```
## ✅ Dry Run Output
If the trap logic triggers, you should see something like:

```bash
shouldRespond: true
responseData: "Your_Discord_Username"
```



