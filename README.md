# Video walkthrough

A video walkthrough of this component can be found here: https://www.youtube.com/watch?v=K1fUcLRBZ2w

## 🧱 Cartesi AVS Operator Setup on Ubuntu 24.04

### 📦 Prerequisites

Ensure the following dependencies are installed:

* **Eigenlayer CLI**
  [Download latest binary from GitHub](https://github.com/Layr-Labs/eigenlayer-cli/releases)

* **Docker + Docker Compose plugin**
  [Install Docker](https://docs.docker.com/engine/install/ubuntu/)

---

### 🗂️ Directory Layout

```bash
sudo mkdir -p /srv/cartesi-operator
cd /srv/cartesi-operator

# Fetch official docker-compose file
curl -O https://raw.githubusercontent.com/cartesi/coprocessor-operator/refs/heads/main/docker-compose.yaml

# Create persistent data directories for one operator instance
mkdir -p operator1-ipfs operator1-snapshots operator1-db
```

---

### 🔑 BLS Key Generation

Generate a fresh BLS key for this AVS:

```bash
~/bin/eigenlayer keys create --key-type bls cartesi-operator
```

> ⚠️ Do not reuse a BLS key from another AVS.
> Note down the BLS private key shown on screen.

---

### 🔐 Store BLS Key as a Docker Secret

```bash
read blskey && echo -n "$blskey" > operator1_bls_private_key && blskey=
```

---

### 🧾 Edit `docker-compose.yaml`

Update the following:

* Set `ETHEREUM_ENDPOINT` in the operator service environment to point to your **Ethereum archival node** endpoint.

Example:

```yaml
environment:
  - ETHEREUM_ENDPOINT=https://mainnet-archive.your-provider.com
```

---

### 🚀 Bring Up Operator Stack

```bash
docker compose up -d --wait
```

---

### 📝 Operator Registration on Holesky (dev)

Prepare registration by running the container manually with volume mounts for deployment info.

#### 1. Organize deployment files

You need the following files:

* `contracts/script/input/holesky_eigenlayer_deployment.json`
* `contracts/script/output/holesky_dev_coprocessor_deployment.json`

(Refer to the [`cartesi/coprocessor`](https://github.com/cartesi/coprocessor) repo for how to generate or fetch these.)

#### 2. Run the setup container

```bash
docker run --rm \
  --name cartesi-coprocessor-setup-operator1 \
  -v $PWD/contracts/script/input/holesky_eigenlayer_deployment.json:/operator/contracts/script/input/holesky_eigenlayer_deployment.json \
  -v $PWD/contracts/script/output/holesky_dev_coprocessor_deployment.json:/operator/contracts/script/output/holesky_dev_coprocessor_deployment.json \
  -e OPERATOR_BLS_KEY="21202655307791246795241072297561426972319640158631289607042740084739930473872" \
  ghcr.io/cartesi/coprocessor-operator:latest \
  bash -c "/operator/setup-operator \
    --el-deployment-file-path contracts/script/input/holesky_eigenlayer_deployment.json \
    --avs-deployment-file-path contracts/script/output/holesky_dev_coprocessor_deployment.json \
    --operator-private-key 0x507dedda46e52e1145b3a81963b6a12b6abf667e85e4eee20b09a2dd294075d2 \
    --operator-socket http://operator1:3033 \
    --el-node-url http://anvil:8545"
```

> ✅ **Notes:**
>
> * `--operator-private-key`: Ethereum key for AVS registration, **not** the BLS key.
> * `--el-node-url`: Any Holesky Ethereum RPC node is fine (no archival requirement).
> * `OPERATOR_BLS_KEY`: Your generated BLS key from above
