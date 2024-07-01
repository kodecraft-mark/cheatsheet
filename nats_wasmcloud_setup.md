# Cluster Set up (NATS, WASMCloud, WADM)

Step by step set up for WASMCloud, WADM, and NATS setup with authentication

## Install NSC tool
1. You can install via nats.io or Github (https://github.com/nats-io/nsc#install)

## Create accounts using NSC tool
1. Create operator
   ```bash
   nsc add operator --generate-signing-key --sys --name <operator_name>
   # Operator account act as your organization (e.g. JABRA)
   # --sys generate system account for full permission
   ```
2. Require signing key and add the NATS url for operator
   ```bash
   nsc edit operator -require-signing-keys --account-jwt-server-url "nats://0.0.0.0:4222"
   # --account-jwt-server-url is a URL where you want to push the JWT
   ```
3. Adding accounts
   ```bash
   nsc add account <ACCOUNT_NAME>
   # Accounts is the group of people that issue different users
   # Such as TEAM_FE, TEAM_BE etc..
   ```
4. Edit account, generate signing keys
   ```bash
   nsc edit account <ACCOUNT_NAME> --sk generate
   ```

5. Adding end users
   ```bash
   nsc add user --account <ACCOUNT_NAME> <USER_NAME>
   # You can check if the user is successfully created using:
   # `nsc list keys -A`
   ```
   - To be able access, the end user need the credential, during creation of user the `user_name.creds` is generated. If you want to pull it out, you can do the below command
     ```bash
     nsc generate creds -n <user_name> > <user_name>.creds
     ```
6. Generate a resolver config file
   ```bash
   nsc generate config --nats-resolver --sys-account SYS > resolver.conf
   # This will generate a nats configuration
   # You can ethier use this configuration as is
   # Or edit it, or import the content into existing configuration
   ```
   - Sample generated `resolver.conf` file
     ```sh
      # Operator named jabra
      operator: <some jwt value>
      # System Account named SYS
      system_account: <some key>

     # Add web socket configuration
      websocket {
        port: 9222
        no_tls: true
      }

     # Add JetStream configuration
      jetstream {}
     
      # configuration of the nats based resolver
      resolver {
          type: full
          # Directory in which the account jwt will be stored
          dir: './jwt'
          # In order to support jwt deletion, set to true
          # If the resolver type is full delete will rename the jwt.
          # This is to allow manual restoration in case of inadvertent deletion.
          # To restore a jwt, remove the added suffix .delete and restart or send a reload signal.
          # To free up storage you must manually delete files with the suffix .delete.
          allow_delete: false
          # Interval at which a nats-server with a nats based account resolver will compare
          # it's state with one random nats based account resolver in the cluster and if needed,
          # exchange jwt and converge on the same set of jwt.
          interval: "2m"
          # Timeout for lookup requests in case an account does not exist locally.
          timeout: "10.0s"
      }
      
      
      # Preload the nats based resolver with the system account jwt.
      # This is not necessary but avoids a bootstrapping system account.
      # This only applies to the system account. Therefore other account jwt are not included here.
      # To populate the resolver:
      # 1) make sure that your operator has the account server URL pointing at your nats servers.
      #    The url must start with: "nats://"
      #    nsc edit operator --account-jwt-server-url nats://localhost:4222
      # 2) push your accounts using: nsc push --all
      #    The argument to push -u is optional if your account server url is set as described.
      # 3) to prune accounts use: nsc push --prune
      #    In order to enable prune you must set above allow_delete to true
      # Later changes to the system account take precedence over the system account jwt listed here.
      resolver_preload: {
              <some_key>: <some_jwt>,
      }
      ```
7. Edit `resolver.conf` file, add your websocket and jetstream configuration or import this on the current configuration as needed
8. Push your changes usng `nsc` command
   ```bash
   nsc push -A
   # Whenever you have changes on operator, accounts, and user settings
   # You need to push the configuration using this command
   # Or if you restart the application
   # You also need to do this, otherwise the application will failed to authenticate
   ```
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

## Reference
1. https://www.youtube.com/watch?v=5pQVjN0ym5w
