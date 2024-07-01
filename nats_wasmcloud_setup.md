# Cluster Set up (NATS, WASMCloud, WADM)

Step by step set up for WASMCloud, WADM, and NATS setup with authentication

## Create account using NSC tool
1. Todo
2. Todo
3. Todo

## NATS (via docker) Set up
1. Run docker set up with the following flags
   ```bash
   docker run --name nats --network nats --rm -p 4222:4222 -p 8222:8222 -p 9222:9222 -v ~/resolver.conf:/etc/nats/resolver.conf nats:latest --http_port 8222 --config /etc/nats/resolver.conf
   ```
2. Todo

## WADM (via docker) Set up
1. Run docker set up with the following flags
   ```bash
   docker run --name wadm --network nats --rm -v ~/DEVOPS.creds:/etc/DEVOPS.creds -e RUST_LOG=debug ghcr.io/wasmcloud/wadm:v0.12.0 --nats-server nats:4222 --nats-creds-file /etc/DEVOPS.creds
   ```
2. Todo
3. Todo

## WASMCloud (via docker) Set up
1. Run docker set up with the following flags
   ```bash
   docker run --name wasmcloud --network nats --rm -v /home/markortal/services-workspace:/media -e "RUST_LOG=debug" -p 4000:4000 -p 4001:4001 -p 8080-8089:8080-8089 -e "WASMCLOUD_RPC_JWT=<jwt>" -e "WASMCLOUD_RPC_SEED=<nkeyseed>" -e "WASMCLOUD_RPC_HOST=nats" -e "WASMCLOUD_CTL_JWT=<jwt>" -e "WASMCLOUD_CTL_SEED=<nkeyseed>" -e "WASMCLOUD_CTL_HOST=nats" -e "WASMCLOUD_PROV_RPC_HOST=nats" -e "OCI_REGISTRY=ghcr.io" -e "OCI_REGISTRY_USER=<github_username>" -e  "OCI_REGISTRY_PASSWORD=<github_token>" -e "WASMCLOUD_RPC_TIMEOUT_MS=10000" -e "WASMCLOUD_ALLOW_FILE_LOAD=true" -e "WASMCLOUD_STRUCTURED_LOGGING_ENABLED=true" ghcr.io/wasmcloud/wasmcloud:canary
   ```
2. Todo
3. Todo
