# 🧩Boundless Prover Guide - The Most Detailed One
The Boundless Prover Node is a computational proving system that participates in the Boundless decentralized proving market. Provers stake USDC bid on computational tasks generate zero-knowledge proofs using GPU acceleration and earn rewards for successful proof generation.

---

## First under how provers work - to know what you are doing : don't skip any line

- Requester posts ZKP request on Boundless (say 10M cycle proof in $5-$10 and 30 min is deadline)
- All provers (say Prover 1-5) see request simultaneously
- Prover 1 locks order at $5 he has to fulfill in 10 mins (LockAndFulfill order)
- Provers 2-5 skip locked order awaiting higher price
  * If Prover 1 attempts proof `Success` Submits in 10 mins, gets paid, no slashing
  * If Failure: Lock expires, Prover 1 slashed

- Post-lock expiry: Order becomes FulfillAfterLockExpire; secondary provers (2-5) attempt proof in remaining 20 mins.
- First secondary prover to submit wins payment; others earn nothing, no slashing.

---

- In all this chaos Boundless has mentioned rewards for primary provers - but i did not come across a single post which says Secondary Provers will be rewarded too - maybe in future they may announce
- Almost all one click nodes (mintair, easy nodes etc) are not able to lock orders even if they do are max secondary orders

---

## Requirements
### Hardware
* CPU - 16 threads, reasonable single core boost performance (>3Ghz)
* Memory - 32 GB
* Disk - 100 GB NVME/SSD
* GPU
  * Minimum: one 8GB vRAM GPU
  * Recommended to be competetive: 10x GPU with min 8GB vRAM
  * Recomended GPU models are 4090, 5090 and L4

### Software
* Supported: Ubuntu 20.04/22.04
* Not supports : Ubuntu 24.04

### Funds
* Have minimum: 2-5 ETH on Base
* Have minimum: $5 USDC on Base
   * note: use burner wallet

---

#### Base Mainnet RPC From : [BlockPI](https://account.getblock.io/sign-up) , [Alchemy](https://www.alchemy.com/) 

---

#### Docker Desktop & Docker compose

* For Local Pc Users: [DockerDesktop](https://www.docker.com/)
  
* #### Install Docker & Docker Compose : Skip If You Don't have Cloud GPU

```
sudo apt update && sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
```

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

```
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```
sudo apt update && sudo apt install -y docker-ce && sudo systemctl enable --now docker
```

```
sudo usermod -aG docker $USER && newgrp docker
```


```
sudo curl -L "https://github.com/docker/compose/releases/download/$(curl -s https://api.github.com/repos/docker/compose/releases/latest | jq -r .tag_name)/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && sudo chmod +x /usr/local/bin/docker-compose
```


*  Verify installation

```
docker --version && docker-compose --version
```





## Install Required Dependecies : For All From Here - **let docker run in background**


```
sudo apt update && sudo apt upgrade -y
```

```
sudo apt install curl iptables build-essential git wget lz4 jq make gcc postgresql-client nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev tar clang bsdmainutils ncdu unzip libleveldb-dev libclang-dev ninja-build -y
```

---

## clone Boundless Repo :

```
git clone https://github.com/boundless-xyz/boundless
```
```
cd boundless && git checkout v0.13.0 && git submodule update --init
```

### install few more dependecies & Tools
>Boundless packages, GPU drivers for provers, Docker with NVIDIA support, Rust, CUDA Toolkit


#### setup Boundless dependencies :

```
sudo ./scripts/setup.sh
```

 ---

### Install Rustup, Cargo, rzup, Rust, cargorisczero : By One click

```
sudo -v
```
```
curl -sL https://raw.githubusercontent.com/morsyxbt/rzup-setup/main/setup.sh | bash
```

### Install :

#### Bento-cli

```
cargo install --locked --git https://github.com/risc0/risc0 bento-client --branch release-2.3 --bin bento_cli
```

```
echo 'export PATH="$HOME/.cargo/bin:$PATH"' >> ~/.bashrc
```

```
source ~/.bashrc
```


#### Boundless CLI :

```
cargo install --locked boundless-cli 
```

```
export PATH=$PATH:/root/.cargo/bin
```

```
source ~/.bashrc
```

#### check version :

```
bento_cli --version 
boundless -h
```


#### just

```
cargo install just
just --version
```

## setup Environment Variables - Wallet & API

```
cd boundless
```

```
sudo curl -o $HOME/boundless/.env https://raw.githubusercontent.com/morsyxbt/boundless-env/main/.env
```

```
sudo nano $HOME/boundless/.env
```

>Replace `Your_ENV_Wallet_PVT_Key_Without_0x` with your wallet Pvt key without 0x

>Replace `Your_RPC_URL` with your Base Mainnet RPC Endpoint don't remove " "

####  Inject .env

```
source .env
```


* >To apply changes You will need to run this command every time you start the prover 

## configuring Your Prover & Broker

#### For single GPU Only :

```
sudo nano compose.yml
```

**IMP Chnages Needed**

* Scroll down to : `x-exec-agent-common: &exec-agent-common` section

  >* `mem_limit:` : write how much ram you can allocate - tip allocate half of what your pc has

  >* `cpus:` : write number of CPU cores you can allocate - tip allocate 50-70% of what you have

* Scroll down to : `gpu_prove_agent0:`
  
  * change `mem_limit:` & `cpus:`  same as above

### configure Segment Size

* change Segment Size in `x-exec-agent-common: &exec-agent-common` according to your `VRAM` - check below table :

    **VRAM  :  Segment Size MAX**

      8GB   :  19
  
      16GB  :  20

      20GB  :  21

      40GB  :  22
  
  * ctrl + o > Y > enter

---

### Now Add Segment Size in the `.env.broker` 

* First copy `.env.broker-template` into `.env.broker`

```
sudo cp .env.broker-template .env.broker
```

* Open and set `Segment Size` same as above step :

```
sudo nano .env.broker
```

* ####  Inject into broker make changes:

```
source .env.broker
```

>note everytime you run prover/broker you will need to inject

---


### Stake In Marketplace:

>deposit some `$USDC` on Base main net in Boundless marketplcae for staking and for locking orders - tip do around $5

* Inject `.env` first:

```
source .env
```

* Add boundless CLI into Path

```
source ~/.bashrc
```

* Deposit Usdc

```
boundless account deposit-stake STAKE_AMOUNT
```

* Verify Stake

```
boundless account stake-balance
```

---

### Run Bento and Benchmarking Bento

 >Bento: `Bento is the local proving infrastructure. Bento will take requests, prove them and return the result.`


1. Here we will run bento & benchmark our GPUs: 

```
sudo just bento
```

* Make Sure your docker is running in Backgroud:

2. Check the logs

```
just bento logs
```


### Benchmarking Bento

>here we will Benchmark our bento to get to know about our `peak_prove_khz` , which we will use in the broker configuration:

What is `peak_prove_khz` ? --This should correspond to the maximum number of cycles per second (in kHz) your proving backend can operate -

* Inject `.env` first:

```
source .env
```

* Benchmark

```
boundless proving benchmark --request-ids {Order_ID}
```

* Replace `{Order_ID)` from the actual previous order id from official Dashboard: [Get_Order_Id](https://explorer.beboundless.xyz/orders)

>You have to use the `Fulfilled` and `Validated` order IDs: (check below Screenshot)

<img width="2184" height="301" alt="image_2025-07-19_18-40-01" src="https://github.com/user-attachments/assets/ce2a7dd9-416d-4a39-b8f5-69385476c130" />



* Look at the below image, U can see `peak_prove_khz = 333` , the prover is estimated to handle ~333,000 cycles per second (~333 khz).

* We will use this in upcoming broker config file: 

* U can test many times to get the accurate of it:

<img width="2559" height="454" alt="Screenshot 2025-07-19 184051" src="https://github.com/user-attachments/assets/df3e91fa-e62d-4d9b-a792-8a7f2b781167" />


* You have to wait until it done: It can take time: Depends on your choosen `Cycles` in the Order: 

---



## Broker Configuration

>The Broker is a service that runs within the Bento proving stack. Responsible for market interactions including bidding on jobs, locking them, issuing job requests to the Bento proving cluster, and submitting proof fulfillments onchain


* Here You will download my updated `Broker.toml` file: But you all need to configure it manually according to your specs and resources: And u have to play with these configuration to make your broker `Competitive`:


1. Install `Broker.toml`

```
cd boundless
sudo wget https://raw.githubusercontent.com/Mayankgg01/Boundless-Prover-Guide/refs/heads/main/broker.toml -O broker.toml
```

2. Open `Broker.toml` 

```
sudo nano broker.toml
```

>Now we will make Changes in it according to our system and resources: 

>❗Mention all the necessary configs which u have to change: Read all them very politely and change your config according to them:


### 1. `mcycle_price`:

* What it means:

   This sets how much you're charging per million cycles (mcycle) when bidding on proving jobs.  This is one of the inputs to decide the minimum price to accept for a request.

* Tip: Decreasing the `mcycle_price` would help your Broker to bid at lower prices for proofs. So Keep it low

* Ex. You can set it like or lower too: "0.000000000000001" 

### 2. `mcycle_price_stake_token`

* What it means:
     
     Defines which token you're pricing in (usdc) . This is used to determine the minimum price to accept an order when paid in staking tokens (usdc).

* Tip: You can reduce it or increse according to u: You can keep as well: `0.000001`

### 3. `priority_requestor_addresses`

* What it means:

     Priority requestor addresses that can bypass the mcycle limit and max input size limit. We alraedy set 4 official `priority_requestor_addresses` in our file:


### 4. `peak_prove_khz`

* What it means:

     This is your GPU's maximum proving speed, in thousands of cycles per second.

* How to set:

>You alraedy test it and got your `peak_prove_khz` here: [Benchmarking Bento](https://github.com/Mayankgg01/Boundless-Prover-Guide?tab=readme-ov-file#benchmarking-bento)

* Edit your `peak_prove_khz` in `broker.toml`

### 5. `max_mcycle_limit` 

* What it means:

     This limits the maximum job size (in mcycles) you’ll accept. `(1 mcycle= 1 million cycles)` Avoids jobs that are too heavy for your hardware. Orders with cycles more than the set parameter will be spikked


* Tip: You have to change this according to your System: Prover with limited resources should reduce this number cause u cant complete this big order: So If your system or u have multiple gpu then set high: like 4000-5000

* 1 GPU users can set it like 1000-2000

### 6. `min_deadline` 

* What it means:

     This is a minimum number of blocks before the requested job expiration that Broker will attempt to lock a job.

>By setting the min deadline, your prover won't accept requests with a deadline less than that.

* How to set:

>Example: min_deadline = 300 means only take jobs that expire in >5 minutes.

>If your prover is fast and stable, you can reduce to 200 to get more jobs.

### 7. `max_concurrent_proofs`

* What it means:

    How many proving jobs your broker can run at the same time. When the numbers of running proving jobs reaches that limit, the system will pause and wait for them to get finished instead of locking more orders. 

* How to set:

>Set this based on your GPU + CPU power.

>Dont set it above 2 if you are on single GPU, If you're running multiple GPUs, you can increase this


### 8. lockin_priority_gas

* What it means:

     Minimum gas fee (in wei) to send priority lock-in txs. Increasing it to consume more gas to outrun other bidders

* How to set:

 >Example: `lockin_priority_gas` = 3000000000000 means 3000 Gwei. (0.000003 eth)

 >This is important for fast inclusion onchain.

 >Keep between 2000-4000 Gwei to stay competitive.

* Web to Calculate it: www.alchemy.com/gwei-calculator


### Check-out official Docs for more info regarding Broker Configuration: [Official-docs](https://docs.beboundless.xyz/provers/broker#settings-in-brokertoml)

---

## Stop the `bento`

>We will stop the `bento` to run the broker & applying config changes:

```
sudo just bento down
```

### Install `foundry` before running the broker

* >It will prevent you from an Error which will come at building broker:

```
sudo curl -L https://foundry.paradigm.xyz | bash -s -- --skip-confirm
```

```
echo 'export PATH="$HOME/.foundry/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

```
foundryup
forge --version
```

```
forge install Arachnid/solidity-stringutils
```

---

## Start The Broker🚀

1. Inject `.env` 

```
source .env
```

2. Run Prover

```
sudo just broker
```

>Running a broker with `just` will also start the `Bento`  through docker compose.

3. Check logs

>If u are doing in Local Pc then check your Docker Desktop that all 11 containers are running: 

>Check the `Broker` containers Logs:

* If cloud GPU then follow:

```
docker ps
```

* Broker Logs

```
just broker logs
```

<img width="2227" height="1256" alt="image" src="https://github.com/user-attachments/assets/c821373d-aacf-473a-80e3-3dc644a4d7c7" />


Here we go🚀......You just have completed the Boundless prover Set-Up: 🥳


---


<div align="center">
