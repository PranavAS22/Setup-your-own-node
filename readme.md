<img width="1700" height="697" alt="Screenshot from 2025-08-18 12-33-22" src="https://github.com/user-attachments/assets/22edefe5-3b12-45d4-a434-45cdb717fc4c" />### Introduction ###
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
  
Explanation:
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

---
## Tring out Besu/teku setup

## Besu Installation (Execution Layer) ##

Update system good pratice 

```sudo apt update```

install java (both teku and besu are based on java)

```sudo apt install openjdk-21-jdk -y```

Note: try to install java 21 beccause i was having error in java 17  
Error:  compiled by a more recent version of the Java Runtime (class file version 65.0), ... recognizes up to 61.0
Error was caused because Besu 25.7.0 was compiled for Java 21

check your install

```java -version```

my version: openjdk version "21.0.8"

install the tar file: https://github.com/hyperledger/besu/releases
change directory to the where the is been downloaded file 

```tar -xvzf besu-<your version here>.tar.gz```

Move to local folder

```sudo mv besu-<your version here> /usr/local/besu```

check the version:

```besu –version```

my version: besu/v25.7.0/linux-x86_64/openjdk-java-21

## Teku Installation (Consensus Clients Layer) ##

Update system good pratice 

```sudo apt update```

install java (both teku and besu are based on java)

```sudo apt install openjdk-21-jdk -y```


check your install

```java -version```

my version: openjdk version "21.0.8"

Install the tar file (teku): https://github.com/ConsenSys/teku/releases
change directory to the where the is been downloaded file

```tar -xvzf teku-<your version here>.tar.gz```

Move to local folder:

```sudo mv teku-<your version here> /usr/local/teku```

Create a shortcut so you can just type "teku"

```sudo ln -s /usr/bin/teku-25.7.1/bin/teku /usr/bin/teku```

 check  the version: 

```teku –version```

My version: teku/v25.7.1/linux-x86_64/-ubuntu-openjdk64bitservervm-java-21

## Setting Up Besu and Teku (Execution + Consensus Clients)

Note i have already made a jwtsecret file if not use command: 

```openssl rand -hex 32 | tr -d "\n" > ~/jwtsecret```


Besu command:

```
besu \
  --data-path="$HOME/besu-data" \
  --network=MAINNET \
  --sync-mode=SNAP \
  --rpc-http-enabled \
  --rpc-http-host=0.0.0.0 \
  --rpc-http-port=8545 \
  --rpc-http-api=ETH,NET,WEB3 \
  --engine-rpc-enabled \
  --engine-rpc-port=8551 \
  --engine-host-allowlist="*" \
  --host-allowlist="*" \
  --engine-jwt-secret="$HOME/jwtsecret
```

Explation:
--data-path="$HOME/besu-data"

Directory where Besu stores blockchain data, keys, and configs.
$HOME/besu-data ensures it’s in your home directory (avoids permission issues).

--network=MAINNET
Connects Besu to Ethereum Mainnet (as opposed to Goerli, Sepolia, or a private chain).

--sync-mode=SNAP
Uses Snap Sync — downloads the latest state snapshot and recent history instead of syncing the entire chain from genesis.
Much faster and uses less storage.

--rpc-http-enabled
Enables the HTTP JSON-RPC endpoint so external apps (e.g., wallets, DApps) can talk to the node.

--rpc-http-host=0.0.0.0
Allows RPC connections from any network interface (not just localhost).

Be careful — this can expose your node publicly.
--rpc-http-port=8545

The port where the HTTP RPC server listens (8545 is the Ethereum default).
--rpc-http-api=ETH,NET,WEB3

Enables specific JSON-RPC APIs:
ETH: Ethereum-specific calls (transactions, blocks, accounts).
NET: Network information (e.g., peer count, chain ID).
WEB3: Node-level info (client version, etc.).

--engine-rpc-enabled
Enables the Engine API, required for communication between execution and consensus clients after the Merge.

--engine-rpc-port=8551
Port for the Engine API server (default 8551).

--engine-host-allowlist="*"
Allows all hostnames to connect to the Engine API.
Should be restricted in production for security.

--host-allowlist="*"
Same concept as above but applies to HTTP RPC connections.

"*" = no restriction.

--engine-jwt-secret="$HOME/jwtsecret"
Path to the JWT secret used to authenticate between execution and consensus clients

Teku command:

```
teku \
  --data-path="$HOME/teku-data" \
  --network=mainnet \
  --ee-endpoint="http://127.0.0.1:8551" \
  --ee-jwt-secret-file="$HOME/jwtsecret" \
  --p2p-enabled=true \
  --rest-api-enabled=true \
  --rest-api-port=5051 \
  --rest-api-host-allowlist="*" \
  --metrics-enabled=true \
  --metrics-port=8008 \
  --checkpoint-sync-url="https://beaconstate-mainnet.chainsafe.io/"
```

Explation:
--data-path="$HOME/teku-data"

Directory where Teku stores beacon chain data, logs, and configurations.
$HOME/teku-data avoids permission issues.

--network=mainnet
Runs Teku on Ethereum’s Mainnet.

--ee-endpoint="http://127.0.0.1:8551"
Specifies the Execution Engine (EE) API endpoint.
This is your execution client (Geth/Besu) running on port 8551.

--ee-jwt-secret-file="$HOME/jwtsecret"
Path to the JWT authentication secret — must match the one used in your execution client so they can talk securely.

--p2p-enabled=true
Enables peer-to-peer networking so Teku can connect to other Ethereum nodes.

--rest-api-enabled=true
Turns on Teku’s REST API for monitoring and interacting with the node programmatically.

--rest-api-port=5051
Port where the REST API will be available.

--rest-api-host-allowlist="*"
Allows REST API access from all hosts.
For security, you’d normally restrict this to localhost in production.

--metrics-enabled=true
Enables a Prometheus metrics endpoint for monitoring.

--metrics-port=8008
Port where Prometheus metrics are served.

--checkpoint-sync-url="https://beaconstate-mainnet.chainsafe.io/"
Uses Checkpoint Sync to start from a recent trusted state instead of syncing from genesis.
This dramatically speeds up the initial sync.

## Tring out Nethermind/Prysm setup

## Nethermind Installation (Execution Layer) ##

Download the zip  file: https://downloads.nethermind.io/

install dependencies:

```sudo apt-get install libsnappy-dev libc6-dev libc6 unzip```

Extract the zip file 
change directory to the where the is been extracted
To run on the mainet: 


```./Nethermind.Runner \
  --config mainnet \
  --Sync.SnapSync true \
  --JsonRpc.JwtSecretFile=$HOME/jwtsecret
```


## Prysm Installation (Consensus Clients Layer) ##


Download the file from: https://github.com/OffchainLabs/prysm/releases

Hhange directry to where you downloaded the fie
make the file executable: 

```chmod +x beacon-chain-v6.0.4-linux-amd64```

Move to local folder:

```mv beacon-chain-v6.0.4-linux-amd64 prysm-beacon```

to run the consense client:

```../prysm-beacon \
  --execution-endpoint=http://127.0.0.1:8551 \
  --jwt-secret=$HOME/jwtsecret \
  --genesis-beacon-api-url=https://beaconstate.info \
  --checkpoint-sync-url=https://beaconstate.info
```

![Screenshot](images/screenshot-2025-08-18.png)


