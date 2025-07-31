<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c91cbb38-97dd-4319-b2eb-462ebdbf263b" />## Why someone need to run a node locally ?

Running an Ethereum node locally lets you access the blockchain directly without relying on third-party services. It gives you full control, better privacy, and the ability to verify data yourself. It's essential for developers, researchers, and anyone who wants to support the Ethereum network or build truly decentralized applications.

## Choosing a snyc method

Before setting up the node, I researched the different types of Ethereum sync modes to choose the one best suited for my project. I selected Snap Sync because it's significantly faster and requires less storage — it downloads a snapshot of the current state instead of the entire chain history.

Here’s a quick comparison of the main sync modes:

| ***Node Type / Sync Mode*** | ***Sync Time***   | ***Disk Space*** | ***Data Stored***                              | ***Security*** | ***Use Case***                                |
|---------------------------|-----------------|----------------|-----------------------------------------------|--------------|---------------------------------------------|
| ***Snap Sync (Full Node)*** | ~2–11 hours     | 200–500 GB     | Latest state, headers, recent blocks (pruned) | ✅ High       | Recommended for most users                  |
| ***Fast Sync (Full Node)*** | ~24+ hours      | 300–600 GB     | Latest state, headers, blocks                 | ✅ High       | Fallback option if snap isn't available     |
| ***Full Node (Pruned)***    | Varies          | 300–600 GB     | Only recent state (~128 blocks)              | ✅ High       | DApp dev, staking                           |
| ***Light Node***            | < 3 hours       | 300–500 MB     | Block headers only                           | ⚠️ Medium     | Mobile or limited-resource systems          |
| ***Archive Node***          | 5–21 days       | 8–12 TB        | Full history and state diffs                 | 🔒 Highest    | Explorers, analytics, indexing              |


I chose Snap Sync for its fast setup, low storage requirement, and high trust level, making it ideal for running a full node locally without needing massive resources.

### offical documentation: https://geth.ethereum.org/docs/fundamentals/sync-modes ###

While exploring how to run an Ethereum node, I came across several clients, but I decided to go with Geth (Go Ethereum) because it is the official Ethereum implementation written in Go and is one of the most widely used. It supports different sync modes, has strong community support, and offers a good balance between performance and features. For my use case, it was the most practical and beginner-friendly option.

for refernces: 
github repo: https://github.com/ethereum/go-ethereum
Go-ethurum offical website: https://geth.ethereum.org/


To set up Geth on my Ubuntu system, I followed the official guide:
https://geth.ethereum.org/docs/getting-started/installing-geth
Note: You can also install it on other OS by following the offical guide.

## Here’s the exact process I followed: ##

### Step 1: Add the official Ethereum PPA repository ###
``` sudo add-apt-repository -y ppa:ethereum/ethereum ```

### Step 2: Update package lists ###

```  sudo apt-get update ``` 

### Step 3: Install the stable version of Geth ###

```  sudo apt-get install ethereum ``` 


Note: You can also install the development version by running sudo apt-get install ethereum-unstable, but I chose the stable version.

### To verify that Geth was successfully installed, I ran: ###

``` geth -v ```

My installed version is:

geth version 1.16.1-stable-12b4131f

