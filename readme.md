## Introduction ##
Running a local Ethereum node provides direct access to the Ethereum network without relying on third-party services. This improves security, decentralization, and reliability when developing, deploying, or interacting with smart contracts. It also enables full control over RPC endpoints, historical data access (depending on the node type), and increased understanding of how Ethereum operates under the hood.

## System Specifications ##
OS: Ubuntu 24.04.2 LTS

RAM: 8 GB

Storage: 512 GB SSD

CPU: 12th Gen Intel® Core™ i3-1215U × 8

## Journey Overview ##
While exploring how to run an Ethereum node, I came across several important details. After Ethereum's transition to Proof of Stake (The Merge), running a full node requires operating two components:

Execution Layer: Handles transactions and smart contract execution (commonly powered by Geth).

Consensus Layer: Maintains the PoS consensus protocol (commonly run using Lighthouse).

Understanding this structure, I decided to begin by setting up the execution layer using Geth, and downloaded Lighthouse for future integration of the consensus layer.

### Geth Installation (Execution Layer) ###
To install Geth on my Ubuntu system, I followed the official documentation and executed the following commands:


3 Enable the official Ethereum Launchpad repository
```sudo add-apt-repository -y ppa:ethereum/ethereum```

# Update package lists
```sudo apt-get update```

# Install the stable version of Geth
```sudo apt-get install ethereum```

Note: To install the development version instead, use:
sudo apt-get install ethereum-unstable

Verify Installation
To ensure Geth was installed successfully, run:

```geth -v```

My installed version: 1.16.1-stable-12b4131f


