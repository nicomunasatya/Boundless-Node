# Boundless Prover Guide

## Boundless Prover market
First, you need to know how **Boundless Prover market** actually works to realize what you are doing.
* **Requester Submits Ask**: A requester (e.g. developer) creates a task or a computation request as an `order` on Boundless, offering funds in ETH or ERC-20 to incentivize participation.
* **Prover Stakes USDC**: The Boundless market requires funds (USDC) deposited as stake before a prover can `bid` on requests.
* **Prover Places Bid**: A prover detects an `order`, submits a `bid`, stating their offered price or resources, which may be lower than the request’s locked funds or other provers’ `bid`s.
* **Prover Locks Order**: If their `bid` is accepted among other provers (e.g., lower `bid`, sufficient stake, or meeting specific criteria), the prover locks the order committing to prove it within a set deadline (`lock-timeout`) using previously staked `USDC`, so other provers can't touch it until it perform computational power.
* **Prover Generates Proof**: The prover completes the task and submits the `proof` to the network.
* **Slashing**: If the `proof` is invalid, incomplete, or the prover fails to deliver (e.g., due to low computational power, malicious behavior or timeout), the slashing mechanism activates, penalizing the prover by forfeiting a part of their staked `USDC` funds.
* **Order Fulfillment**: If the `proof` is valid, the prover receives the locked funds as a reward, and the requester receives the verified result, completing the process.
* `bid ` are actually `mcycle_price` (the price of each 1 million cycles the prover proves). I'll tell you more about this later in the guide.

---

## Notes
- The prover is in beta phase, while I admit that my guide is really perfect, you may get some troubles in the process of running it, so you can wait until official incentivized testnet with more stable network and more updates to this guide, or start exprimenting now.
- I advice to start with testnet networks due to loss of stake funds
- I will update this github guide constantly, so you always have to check back here later and follow me on [X](https://x.com/0xMoei) for new updates.

---

## Requirements
### Hardware
* CPU - 16 threads, reasonable single core boost performance (>3Ghz)
* Memory - 32 GB
* Disk - 100 GB NVME/SSD
* GPU
  * Minimum: one 8GB vRAM GPU
  * Recommended to be competetive: 10x GPU with min 8GB vRAM
  * Recomended GPU models are 4090, 5090 and L4.
> * I've tested the new release with only a 80GB vRAM GPU, I'll update here when I just tested out lower GPUs.
> * You better test it out with single GPUs by lowering your configurations later by reading the further sections.

### Software
* Supported: Ubuntu 20.04/22.04
* No support: Ubuntu 24.04
* If you are running on Windows os locally, install Ubuntu 22 WSL using this [Guide](https://github.com/0xmoei/Install-Linux-on-Windows)

---

## Rent GPU
* **Beginner Guide**
  * For those new to renting GPUs, refer to [this guide](https://github.com/0xmoei/Rent-and-Config-GPU) for step-by-step instructions.

* **Choosing the Right GPU Template**
  * Rent an `Ubuntu VM` template for your GPU instance.
  * Avoid `CUDA` or `Pytorch` templates, as they are incompatible with the Prover installation.

* **Why Ubuntu VM is Required**
  * Prover installation uses `Docker`. Since `CUDA` or `Pytorch` templates run your GPU instance inside a Docker container, you cannot run the Prover Docker inside another Docker instance.

* **Recommended GPU Providers**
  * **[Vast.ai](https://cloud.vast.ai/?ref_id=62897&creator_id=62897&name=Ubuntu%2022.04%20VM)**: Affordable GPU provider supporting `Ubuntu VM` templates with crypto payment options.
  
---

# Setup
Here is the step by step guide to Install and run your Prover smoothly, but please pay attention to these notes:
* Read every single word of this guide, if you really want to know what you are doing.
* There is an [Prover+Broker Optimization](https://github.com/0xmoei/boundless/blob/main/README.md#bento-prover--broker-optimizations) section where you need to read after setting up prover.

## Dependecies
### Install & Update Packages
```bash
apt update && apt upgrade -y
apt install curl iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev tar clang bsdmainutils ncdu unzip libleveldb-dev libclang-dev ninja-build -y
```

### Clone Boundless Repo
```bash
git clone https://github.com/boundless-xyz/boundless
cd boundless
git checkout release-0.10
```

### Install Dependecies
To run a Boundless prover, you'll need the following dependencies:
* Docker compose
* GPU Drivers
* Docker Nvidia Support
* Rust programming language
* `Just` command runner
* CUDA Tollkit

For a quick set up of Boundless dependencies on Ubuntu 22.04 LTS, you can run:
```bash
bash ./scripts/setup.sh
```
However, we need to install some dependecies manually:

```console
\\ Execute command lines one by one

# Install rustup:
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
. "$HOME/.cargo/env"

# Update rustup:
rustup update

# Install the Rust Toolchain:
apt update
apt install cargo

# Verify Cargo:
cargo --version

# Install rzup:
curl -L https://risczero.com/install | bash
source ~/.bashrc

# Verify rzup:
rzup --version

# Install RISC Zero Rust Toolchain:
rzup install rust

# Install cargo-risczero:
cargo install cargo-risczero
rzup install cargo-risczero

# Update rustup:
rustup update

# Install Bento-client:
cargo install --git https://github.com/risc0/risc0 bento-client --bin bento_cli
echo 'export PATH="$HOME/.cargo/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Verify Bento-client:
bento_cli --version

# Install Boundless CLI:
cargo install --locked boundless-cli
export PATH=$PATH:/root/.cargo/bin
source ~/.bashrc

# Verify boundless-cli:
boundless -h
```

---

## System Hardware Check
* In the beginning, to configure your Prover, You need to know what's your GPUs IDs (if multiple GPUs), CPU cores and RAM.
* Also the following tools are best to monitor your hardware during proving.

### GPU Check:
* If your Nvidia driver and Cuda tools are installed succesfully, run the following command to see your GPUs status:
```
nvidia-smi
```
* You can now monitor Nvidia driver & Cuda Version, GPU utilization & memory usage.
* In the image below, there are four GPUs with **0-3** IDs, you'll need it when adding GPU to your configuration.

![image](https://github.com/user-attachments/assets/26c57f43-0fbf-4068-949c-b2ea31273998)

* Check your system GPUs IDs (e.g. 0 through X):
```bash
nvidia-smi -L
```

### Number of CPU Cores and Threads:
```
lscpu
```
### CPU & RAM check (Realtime):
To see the status of your CPU and RAM.
```bash
htop
```

### GPU Check (Realtime):
The best for real-time monitoring your GPUs in a seprated terminal while your prover is proving.
```bash
nvtop
```

---

## Configure Prover
### Single GPU:
The default `compose.yml` file defines all services within Prover.
* Default `compose.yml` only supporting single-GPU and default CPU, RAM utilization.
* Edit `compose.yml` by this command:
  ```bash
  nano compose.yml
  ```
* The current `compose.yml` is set for `1` GPU by default, you can bypass editing it if you only have one GPU.
* In single GPUs, you can increase the RAM & CPU of the `x-exec-agent-common` and `gpu_prove_agent0` services in `compose.yml` instead to maximize the utilization of your system

### Multiple GPUs
* 4 GPUs:
To add more GPUs or modify CPU and RAM sepcified to each GPU, replace the current compose file with my [custom compose.yml](https://github.com/0xmoei/boundless/blob/main/compose.yml) with 4 custom GPUs

* More/Less than 4 GPUs:
Follow this [detailes step by step guide](https://github.com/0xmoei/boundless/blob/main/add-remove-gpu.md) to add or remove the number of 4 GPUs in my custom `compose.yml` file

### Modify CPU/RAM of x-exec-agent-common
* `x-exec-agent-common` service in your `compose.yml` is doing the preflight process of orders to estimate if prover can bid on them or not.
* More exec agents will be able to preflight and prove more orders concurrently.
* Increasing is from default value: `2` depends on how many concurrent proofs you want to allow.
* You see smth like below code as your `x-exec-agent-common` service in your `compose.yml` where you can increase it's memory and cpu cores:
```yml
x-exec-agent-common: &exec-agent-common
  <<: *agent-common
  mem_limit: 4G
  cpus: 3
  environment:
    <<: *base-environment
    RISC0_KECCAK_PO2: ${RISC0_KECCAK_PO2:-17}
  entrypoint: /app/agent -t exec --segment-po2 ${SEGMENT_SIZE:-21}
```

### Modify CPU/RAM of gpu_prove_agent
* `gpu_prove_agent` service in your `compose.yml` handles proving the orders after they got locked by utilizing your GPUs.
* In single GPUs, you can increase performance by increasing CPU/RAM of GPU agents.
* The default number of its CPU and RAM is fine but if you have good system spec, you can increase them for each GPU.
* You see smth like below code as your `gpu_prove_agentX` service in your `compose.yml` where you can increase the memory and cpu cores of each gpu agent.
   ```yml
     gpu_prove_agent0:
       <<: *agent-common
       runtime: nvidia
       mem_limit: 4G
       cpus: 4
       entrypoint: /app/agent -t prove
       deploy:
         resources:
           reservations:
             devices:
               - driver: nvidia
                 device_ids: ['0']
                 capabilities: [gpu]
   ```
* While the default CPU/RAM for each GPU is enough, for single GPUs, you can increase them to increase efiiciency, but don't maximize and always keep some CPU/RAML for other jobs.

---

**Note**: `SEGMENT_SIZE` of the prover is set to `21` by default in `x-exec-agent-common` service which applies on all GPUs, and `21` is compatible only with `>20GB vRAM` GPUs, if you have less vRAM, you have to modify it, you can read the [Segment Size](https://github.com/0xmoei/boundless/tree/main#segment-size-prover) section of the guide to modify it.

---

## Running Prover
Boundless is comprised of two major components:
* `Bento` is the local proving infrastructure. Bento will take the locked orders from `Broker`, prove them and return the result to `Broker`.
* `Broker` interacts with the Boundless market. `Broker` can submit or request proves from the market.

---

## Run Bento
To get started with a test proof on a new proving machine, let's run `Bento` to benchmark our GPUs:
```bash
just bento
```
* This will spin up `bento` without the `broker`.

Check the logs :
```bash
just bento logs
```

Run a test proof:
```bash
RUST_LOG=info bento_cli -c 32
```
* If everything works well, you should see something like the following as `Job Done!`:

![image](https://github.com/user-attachments/assets/a67fdfb0-3d22-4a4a-b47a-247567df0d45)

* To check if all your GPUs are utilizing:
  *  Increase `32` to `1024`/`2048`/`4096`
  *  Open new terminal with `nvtop` command
  *  Run the test proof and monitor your GPUs utilization.

---
