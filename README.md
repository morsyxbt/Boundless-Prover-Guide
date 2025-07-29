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
>remove STAKE_AMOUNT from above command and enter how much USDC you want to stake
>note: just write : 2 , 3 or 5 don't put any $ sign

* Verify Stake

```
boundless account stake-balance
```

---

### Run Bento and Benchmarking Bento

```
sudo just bento
```

* Your Docker should be running in background
* may take upto 10 minutes

* check logs

```
just bento logs
```


### Benchmark Bento

>Benchmark our bento to know about `peak_prove_khz` , which we will use in the broker configuration:
> This corresponds to the maximum number of cycles per second (in kHz) that your proving backend can operate

* Inject .env first:

```
source .env
```

* Benchmark
* 
* visit : [Order_Id](https://explorer.beboundless.xyz/orders)
* Grab a fullfilled order id - but conditions :
 >order id should have : 10M-50M cycles
 >order id should be : atleast 30minutes-1.5hours old
 >if there's none availabe then you can grab : 100M cycles - check ss

![WhatsApp Image 2025-07-29 at 18 57 08_539d0929](https://github.com/user-attachments/assets/55e550d6-aefe-4378-954d-fc757f7f1bb8)

### Now run :
```
boundless proving benchmark --request-ids {Order_ID}
```
>Replace `{Order_ID)` from the actual previous order id you got above

* You will see `peak_prove_khz = xxx` , the prover is estimated to handle ~xxx,xxx cycles per second (~xxx khz)
  >write the number `xxx` or whatever you get somewhere
  >repeat this benchmark test atleast 3 times with different order id's to get the average number
  
* >`peak_prove_khz` will look like this :
  ![WhatsApp Image 2025-07-29 at 19 05 02_a6920711](https://github.com/user-attachments/assets/73af6f1c-180a-4499-824a-7292eaeaffff)

**This was a test to get our average peak prove khz which we will use in Broker Configuration**

---

## Now Broker Configuration :

* Download my updated `Broker.toml` file : But configure it manually according to your specs and resources
  >all the configurations from here will decide whether you can lock orders and be competitive or not
  >so read every detail properly from here - i have added tips accordingly, you can use them or modify them if you want

* First Install `Broker.toml`

```
cd boundless
sudo wget https://raw.githubusercontent.com/Mayankgg01/Boundless-Prover-Guide/refs/heads/main/broker.toml -O broker.toml
```

2. Open `Broker.toml` 

```
sudo nano broker.toml
```

>Now make changes according to your specs and resources :

### 1. mcycle_price :
   
* defines the price you're bidding per million cycles (mcycle) when proving. This value helps determine the minimum price you're willing to accept for a job

* Tip : Lowering this value allows your broker to bid more competitively
* You can set it to something like "0.000000000000001" or even lower

### 2. mcycle_price_stake_token :

* specifies the token (e.g., usdc) used to calculate pricing for orders paid in staking tokens

* Tip : Adjust this up or down depending on your strategy
* Example : You can keep it like: 0.000001

### 3. priority_requestor_addresses :

* A list of requestor addresses that bypass the mcycle and input size limits
* I have already included four official `priority_requestor_addresses` in this config


### 4. peak_prove_khz :

* Represents your GPU's maximum proving speed measured in thousands of cycles per second (kHz)

* How to set :
>You alraedy tested it and got your `peak_prove_khz` average above : Benchmarking Bento

* Now edit your `peak_prove_khz` in `broker.toml` as per the average you got above
  
### 5. max_mcycle_limit :

* Controls the maximum maximum job size (in mcycles) you’ll accept `(1 mcycle= 1 million cycles)` this helps avoid assigning jobs too large for your hardware

* Tip:
  * If you're on a single GPU, keep it around `1000–2000`
  *If you have multiple GPUs, set it higher, like 4000–500

>change this according to your system - prover with limited resources should reduce this number otherwise you won't complete big orders

### 6. min_deadline :

* Minimum number of blocks before a job expires, below which the broker won’t attempt to lock the job

* Example : `min_deadline = 300` means your broker will only accept jobs that expire in more than 5 minutes

* Tip : If your setup is fast and reliable, consider lowering this to 200 to grab more jobs

### 7. max_concurrent_proofs :

* Controls how many jobs your broker can prove simultaneously
* If the number of running proofs reaches this cap the broker will pause before locking new ones

* Tip : On single-GPU systems keep this at `2` or less
* For multi-GPU setups, you can raise it


### 8. lockin_priority_gas :

* Minimum gas fee (in wei) used for priority lock-in transactions. Higher values help you outbid others and get your lock-in confirmed faster

* Example : `lockin_priority_gas` = 3000000000000 means 3000 Gwei (≈ 0.000003 ETH)

* Tip : Keep it between 2000–4000 Gwei for better on-chain competition

* Here's a web to calculate it : [GWEI_Calculator](www.alchemy.com/gwei-calculator)

---

## Stop `bento` :

>stop `bento` to run the broker & applying config changes :

```
sudo just bento down
```

### Install foundry before running the broker :

>This prevents from an error which comes at building broker

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

## Now start Broker :

* Inject .env 

```
source .env
```

* Run Prover

```
sudo just broker
```

>Running a broker with `just` can also start the `Bento`  through docker compose

3. Check logs

>If Local Pc then check your Docker Desktop that all 11 containers are running

>Check the `Broker` containers Logs

* **If cloud GPU :**

```
docker ps
```

* Broker Logs

```
just broker logs
```

* Boom congrats : you completed Boundless Prover setup 

---

### some important points and solutions of errors :

### 3. understand to identify primary/secondary transactions :


* >Fullfilled Transaction Primary Prover : `0x8e3b6945`
  
* >Lock Order Successfully : `0xb4206dd2`

* >Complete Order As Secondry Prover : `0x2f13a90a`

### everything about primary/secondary provers is discussed in the beginning of guide make sure to read that also :
>and reminding again : Boundless has mentioned rewards for primary provers - but i did not come across a single post which says Secondary Provers will be rewarded too - they may or may not announce in future

* **Explorer: https://basescan.org/address/**

---

### 2. solution of : error the following required arguments were not provided --rpc-url <RPC_URL>

>This error pops due to env file not etting injected into broker/bento or docker compose file sets to use env in place of env.base


 #### Follow the commands :
 
 * Down broker/bento :

```
sudo just bento down
```

```
sudo just broker down
```

#### move .env.base into .env

```
mv .env.base .env
```


#### Reload bash

```
source .env
```


#### start bento/broker

```
sudo just bento 
```

```
sudo just broker 
```

* This should fix the error

---

### 3. If your local pc shuts down/next day start :

* Move to boundless Directory :
  
```
cd boundless
```

* start your docker desktop

* Inject .env and .env.broker into broker :


```
source .env.broker
```

```
source .env
```


#### Start broker :

```
sudo just broker
```

* check your containers and logs
* done

---

**Made with ❤️ by [Morsyxbt](https://x.com/morsyxbt)**

---

**A note in the end :**

>This took a lot of hardwork analysing docs not only that - i had to checek almost 10+ other guides to see what's going wrong why aren't people being able to lock primary orders
>max guides were not even understandable for non-coding background people

>shoutout to all those people that had put in the work even before me - they were a huge help
>with some time i will be adding more accurate configurations don't worry
>boundless is truly one of the tough nodes out there - you are competing against devs who have maybe 5-10x of resources
>so be glad if you will be able to lock orders against them
>you will learn a lot in this journey
>peace out
