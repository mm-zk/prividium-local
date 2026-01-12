# prividium-local

Setting up prividium locally.

warning - you'll need access to some internal binaries to finish this.

## Getting zksync os to run.

```shell
# Basics
sudo apt-get update
sudo apt-get install -y git tmux build-essential pkg-config cmake clang lldb lld libssl-dev libpq-dev apt-transport-https ca-certificates curl software-properties-common nginx

# Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
sudo apt install -y docker-ce
sudo usermod -aG docker ${USER}

# Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Anvil
curl -L https://foundry.paradigm.xyz | bash
source ~/.bashrc
foundryup -i 1.3.4

# code
git clone https://github.com/matter-labs/zksync-os-server.git



# Run (each one on separate screen)
anvil --load-state ./local-chains/v31/zkos-l1-state.json --port 8545
cargo run -- --config ./local-chains/v31/multiple-chains/chain1.json
cargo run -- --config ./local-chains/v31/multiple-chains/chain2.json
```

## Docker images


Run 'docker save' on the machine that had access to internal repo.

```shell
docker save -o prividium-compose-images.tar \
  zksync-prividium-dual-admin-panel-l2a:latest \
  zksync-prividium-dual-admin-panel-l2b:latest \
  ghcr.io/matter-labs/block-explorer-api:latest \
  ghcr.io/matter-labs/block-explorer-app:latest \
  ghcr.io/matter-labs/block-explorer-data-fetcher:latest \
  ghcr.io/matter-labs/block-explorer-worker:latest \
  zksync-prividium-dual-permissions-api-l2a:latest \
  zksync-prividium-dual-permissions-api-l2b:latest \
  zksync-prividium-dual-proxy-l2a:latest \
  zksync-prividium-dual-proxy-l2b:latest \
  zksync-prividium-dual-user-panel-l2a:latest \
  zksync-prividium-dual-user-panel-l2b:latest \
  quay.io/keycloak/keycloak:26.0 \
  postgres:15
```

Copy prividium-compose-images.tar to the destination machine.

```shell
docker load -i prividium-compose-images.tar
```

## Configuring ports and settings.


### HTTPS

Prividium requires HTTPS to work. Here we'll assume that you create self-signed certs, and don't have DNS (this will require 'accepting' some warnings later in the browser). 

Create local certs

```shell
sudo mkdir -p /etc/nginx/certs
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/nginx/certs/privkey.pem \
  -out /etc/nginx/certs/fullchain.pem
```

Copy the config

```shell
sudo cp https-ports.conf /etc/nginx/conf.d
sudo systemctl reload nginx
```


### Setting ports and permissions

Now we have to do a few replacements:


Replace http://localhost with https://YOUR-IP
* in docker-compose-dual.yaml
* in dev/keycloak/realm-export.json
* in dev/block-explorer JSON files.


### Starting

```
docker compose  -f docker-compose-dual.yaml  up --no-build --pull never
```

### Permissions

Your machine must allow access to following ports:

* 3000,3001,3010,8000,8001
* 3300,3301,3310,8300,8301
* 5080 (keycloak)


## Trying it out

Now you can login to admin panel of the first chain (port 3000) or second chain (port 3300).


You might want to use Google Chrome - as Firefox complains a lot about the invalid certificates.
