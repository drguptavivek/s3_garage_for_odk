# Garage S3 Setup Instructions

This directory contains the configuration to set up a Garage S3 server using Docker, designed to sit behind an Nginx reverse proxy.

## 1. Generate Secrets

Before starting, you need to generate secure random strings for the configuration.

Run the following commands to generate secrets:

```bash
# Generate RPC Secret
openssl rand -hex 32

# Generate Admin Token
openssl rand -base64 32

# Generate Metrics Token
openssl rand -base64 32
```

## 2. Configure `garage.toml`

Open `garage.toml` and replace the placeholders:

- `rpc_secret`: Replace `REPLACE_WITH_YOUR_RPC_SECRET` with the hex string generated above.
- `admin_token`: Replace `REPLACE_WITH_YOUR_ADMIN_TOKEN` with the base64 string.
- `metrics_token`: Replace `REPLACE_WITH_YOUR_METRICS_TOKEN` with the second base64 string.
- `root_domain`: Change `.s3.domain.tld` and `.web.domain.tld` to your actual domain.
- `rpc_public_addr`: Defaults to `127.0.0.1:3901`. If you plan to add more nodes or if this IP isn't reachable, change it to your VM's public or LAN IP.

## 3. Start Garage

Run the container using Docker Compose:

```bash
docker compose up -d
```

## 4. Initial Setup (Layout)

Since this is a new cluster, you need to initialize the layout.

1. Get the node ID:
   ```bash
   docker compose exec garage garage node id
   ```

2. Create the layout with the single node (replace `<NODE_ID>` with the ID from the previous step).
   *Note: `-c 1G` sets the capacity to 1GB. Adjust this to the size of your allocated volume (e.g., `100G`).*
   ```bash
   docker compose exec garage garage layout assign -z zone1 -c 100G <NODE_ID>
   docker compose exec garage garage layout apply --version 1
   ```

## 5. Nginx Configuration

An example Nginx configuration is provided in `nginx_upstream_example.conf`. You should incorporate this into your upstream Nginx server's configuration.

**Important:** In `nginx_upstream_example.conf`, the upstream server is defined as `server 127.0.0.1:3900;`. If your Nginx proxy is on a different machine than the Garage VM, you must replace `127.0.0.1` with the IP address of the Garage VM.

Ensure your DNS is set up correctly:
- `s3.domain.tld` -> IP of Nginx Proxy
- `*.s3.domain.tld` -> IP of Nginx Proxy

## 6. Verification

You can check the status of the garage node:

```bash
docker compose exec garage garage status
```
