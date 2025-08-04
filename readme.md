### Introduction ###
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

## Geth Installation (Execution Layer) ##
To install Geth on my Ubuntu system, I followed the official documentation and executed the following commands:


# Enable the official Ethereum Launchpad repository
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

## Setting Up Geth and Lighthouse (Execution + Consensus Clients)
# Downloading and Moving Lighthouse
I started by downloading the Lighthouse binary as a .tar.gz file and then extracted and moved it to the system's executable path:


```
 tar -xvf lighthouse-v*.tar.gz
sudo mv lighthouse /usr/bin/
```
This made the lighthouse command globally accessible on my system.

# Running Geth (Execution Client)
Next, I referred to the official Ethereum documentation to start Geth. Initially, I tried this:

```
geth --mainnet \
  --datadir "/data/ethereum" \
  --http \
  --authrpc.addr localhost \
  --authrpc.vhosts="localhost" \
  --authrpc.port 8551 \
  --authrpc.jwtsecret=~/jwtsecret
```
  
🔍 Explanation:
--mainnet: Connects to the Ethereum mainnet.

--datadir: Specifies the data directory for storing blockchain data.

--http: Enables the HTTP-RPC server.

--authrpc.addr: Binds the Auth RPC endpoint to localhost.

--authrpc.vhosts: Sets allowed virtual hosts.

--authrpc.port: Port for the Auth RPC.

--authrpc.jwtsecret: Path to the JWT secret used to communicate securely with the consensus client.

I also created the JWT secret file using:
```openssl rand -hex 32 | tr -d "\n" > ~/jwtsecret```

# First Error Faced
When I ran the command, I faced the following errors:

Smartcard socket not found, disabling    err="stat /run/pcscd/pcscd.comm: no such file or directory"

Fatal: Failed to create the protocol stack: mkdir /data: permission denied

bash: --authrpc.jwtsecret=~/jwtsecret: No such file or directory
What went wrong?

Geth couldn’t access the /data directory (due to lack of permission).

The ~ shortcut for home wasn’t working as expected in --authrpc.jwtsecret.

#Fixing the Geth Command
   
So, I fixed both issues by:

Switching to a directory in my home path.

Replacing ~ with $HOME.

Here's the updated working version:


```
geth --mainnet \
  --syncmode "snap" \
  --datadir "$HOME/ethereum-data" \
  --http \
  --authrpc.addr localhost \
  --authrpc.vhosts="localhost" \
  --authrpc.port 8551 \
  --authrpc.jwtsecret="$HOME/jwtsecret"
```
Even though snap is the default sync mode, I explicitly included it to avoid ambiguity. Also, using $HOME ensures the full path is correctly resolved.

#Setting Up Lighthouse (Consensus Client)
Then, I moved on to Lighthouse and ran:

```
lighthouse beacon_node \
  --network mainnet \
  --datadir /data/ethereum \
  --http \
  --execution-endpoint http://127.0.0.1:8551 \
  --execution-jwt ~/jwtsecret
```
But again I faced permission issues:


Failed to initialize rolling file appender: Failed to create directory '/data/ethereum/beacon/logs': Permission denied

bash: --authrpc.jwtsecret=~/jwtsecret: No such file or directory

So again, same issue: permission problems with /data, and the ~ path not being resolved.


# Final Working Lighthouse Command
After fixing the directory and JWT path, and adding the checkpoint sync URL (to avoid syncing from genesis, which is insecure), here’s my current command:

```
lighthouse beacon_node \
  --network mainnet \
  --datadir "$HOME/ethereum-data" \
  --http \
  --execution-endpoint http://127.0.0.1:8551 \
  --execution-jwt "$HOME/jwtsecret" \
  --checkpoint-sync-url https://mainnet-checkpoint-sync.attestant.io/
```
  
# Current Error: Checkpoint Sync DNS Failure
Right now, I'm stuck with this error:

Failed to start beacon node reason: "Error loading checkpoint state from remote: HttpClient(url: https://checkpoint-sync.ethpandaops.io/, kind: request, detail: error trying to connect: dns error: 

failed to lookup address information: No address associated with hostname)"

This seems like a DNS resolution issue when trying to reach the checkpoint sync URL. I suspect it’s either:
A temporary internet issue.
DNS misconfiguration.
Or the endpoint is down/unavailable.
I plan to try other recommended checkpoint endpoints until I find one that connects smoothly.


