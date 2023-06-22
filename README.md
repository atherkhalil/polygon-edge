# Polygon Edge - Production Deployment Guide

This document lists out the commands used for deploying a production grade Polygon Edge network with secure secrets.

## Clone & Build

```bash
git clone https://github.com/0xPolygon/polygon-edge.git

make build
```

## Create A Key Vault

The following is a docker-compose file used to create a docker container for the Key Vault (Hashicorp) to store node secrets. The vault is exposed on `http://127.0.0.1:8200`.

```bash
version: '3'
services:
  myvault:
    image: vault
    container_name: myvault
    ports:
      - 8200:8200
    environment:
       VAULT_SERVER: "http://127.0.0.1:8200"
       VAULT_DEV_ROOT_TOKEN_ID: "my-token"
```

## Create Secrets

Now that the Key Vault is accessible, we'll supply the url & token to the "init" command to store the secrets in it after creation.

```bash
./polygon-edge secrets generate --name secretMan --token my-token --server-url http://127.0.0.1:8200
```

```bash
./polygon-edge secrets init --config ./secretsManagerConfig.json
```

#### The above commands will store the secrets in the specified Key Vault and print the following.

```bash
[SECRETS INIT]
Public key (address) = 0xf0b581F4256B8801D8e397a0024883eeBdEe2e38
BLS Public key       = 0x87b756961fa6304bfcf5156177a782e22f0b077ad2bef01f0b175a76ca4928fd0637704fe724073cd64dbd2c919d0ba8
Node ID              = 16Uiu2HAm6CVzf6VfHqR5WnFwZCdBrKiGaTsqU2McXBVTjqfzUTe7
```

## Create Genesis File

```bash
./polygon-edge genesis --consensus ibft --ibft-validator 0xf0b581F4256B8801D8e397a0024883eeBdEe2e38:0x87b756961fa6304bfcf5156177a782e22f0b077ad2bef01f0b175a76ca4928fd0637704fe724073cd64dbd2c919d0ba8 --bootnode /ip4/127.0.0.1/tcp/10001/p2p/16Uiu2HAm6CVzf6VfHqR5WnFwZCdBrKiGaTsqU2McXBVTjqfzUTe7
```

## Start Node

```bash
./polygon-edge server --data-dir chain-data --secrets-config ./secretsManagerConfig.json --chain ./genesis.json --grpc-address :10000 --libp2p :30301 --jsonrpc :10002 --seal
```
