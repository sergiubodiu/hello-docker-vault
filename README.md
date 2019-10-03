# Getting Started with Docker on Vault

## Setup Hashicorp Vault Server on Docker

### Create the directory structure

```bash
mkdir -p volumes/{config,file,logs}
```

Populate the vault config `vault.json`

```bash
cat > volumes/config/vault.json << EOF
{
  "backend": {
    "file": {
      "path": "/vault/file"
    }
  },
  "listener": {
    "tcp":{
      "address": "0.0.0.0:8200",
      "tls_disable": 1
    }
  },
  "ui": true
}
EOF
```

Populate the `docker-compose.yml`

```bash
cat > docker-compose.yml << EOF
version: '2'
services:
  vault:
    image: vault
    container_name: vault
    ports:
      - "8200:8200"
    restart: always
    volumes:
      - ./volumes/logs:/vault/logs
      - ./volumes/file:/vault/file
      - ./volumes/config:/vault/config
    cap_add:
      - IPC_LOCK
    entrypoint: vault server -config=/vault/config/vault.json
EOF
```

Start the Vault Server:

```bash
docker-compose up
```

The UI is available at http://localhost:8200/ui and the api at http://localhost:8200

## Getting Started CLI Guide

Installing the vault cli tools (Mac):

```bash
brew install vault

# Set environment variables:
export VAULT_ADDR='http://127.0.0.1:8200'

```

### Initialize new vault cluster with 5 key shares

```bash
vault operator init -key-shares=5 -key-threshold=3
Unseal Key 1: okFz8kYG3Lr7+VC5WHp9FpxbluZCK+RLB2si7peBfdCx
Unseal Key 2: tBApy7ogda2eqoHbMenyTlVOuJQW4bdK7UY+WZA4H5Pw
Unseal Key 3: 4qk2hHGqOJRZ5zkItRVqcLrKuOT1ZbvbhO8rwNiJKk6l
Unseal Key 4: /2lpJTuCU6v8iZgzWEh1aYM62CpdoW/DQp8/JVJLXXFh
Unseal Key 5: Vynka40xCwQ2kdS+KXUafsQZkK21ofB54CsbfeqdmOkJ

Initial Root Token: s.X3FHS7YTB3VPSQjSpQUBqaTE

Vault initialized with 5 key shares and a key threshold of 3. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, you must supply at least 3 of these keys to unseal it
before it can start servicing requests.

Vault does not store the generated master key. Without at least 3 key to
reconstruct the master key, Vault will remain permanently sealed!

It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.
```

### Ensure the vault is unsealed

In order to unseal the vault cluster, we need to supply it with 3 key shares `vault operator unseal`:

```bash
vault status -format=json
{
  "type": "shamir",
  "initialized": true,
  "sealed": false,
  "t": 3,
  "n": 5,
  "progress": 0,
  "nonce": "",
  "version": "1.2.3",
  "migration": false,
  "cluster_name": "vault-cluster-b94f7bf1",
  "cluster_id": "d22810aa-0637-d552-5558-9cfd69671ab3",
  "recovery_seal": false
}
```
