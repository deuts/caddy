# Custom Caddy Docker Image with Cloudflare DNS Challenge Support

This repository provides a Dockerfile that builds a custom version of the [Caddy](https://caddyserver.com/) web server, with support for the Cloudflare DNS challenge. This setup is particularly useful for automatically obtaining and renewing SSL/TLS certificates using Cloudflare’s DNS API.

## Why Use This?

When using Caddy's automatic HTTPS capabilities, the HTTP and TLS challenges are sufficient for most use cases. However, if your server is behind a firewall, CGNAT, load balancer, or you don't want to expose port 80/443 to the public internet, the **DNS challenge** is a better option. This Dockerfile extends the official Caddy image by integrating the `xcaddy` tool to build Caddy with the `cloudflare` DNS provider module.

With this setup, you can use Cloudflare's API to verify domain ownership and automatically obtain certificates for your domain(s).

## How It Works

The Dockerfile is composed of two stages:

1. **Builder Stage**: The Caddy executable is built using `xcaddy`, with the Cloudflare DNS module added as a plugin.
2. **Final Stage**: The custom-built Caddy binary is copied into a fresh Caddy image to ensure a lean and production-ready image.

## Usage

### Prepare

#### Create a caddynetwork
This is optional, but I always liked to create a custom Docker bridge network so I don't have to expose my services' ports to the host, especially if I'm running this on a VPS with a public IP address:
```bash
docker network create caddynetwork
```
If you don't want to create a caddynetwork, or you're using a different name, be sure to make the necessary changes in the `compose.yml` file.

#### Clone the repository
```bash
git clone https://github.com/deuts/caddy.git
```

### Edit `compose.yml`
Rename `_compose.yml` to `compose.yml` and make the necessary changes to the environments:
```yml
    environment:
      - CLOUDFLARE_API_TOKEN=<ENTER YOUR TOKEN HERE>
      - CADDY_ACME_EMAIL=<ENTER YOUR EMAIL HERE>
```

- `CLOUDFLARE_API_TOKEN`: This is your Cloudflare API token. It must have permissions for DNS zone edits.

### Build and run
```bash
docker compose up -d --build
```

### Edit Caddyfile
Make the necessary changes to the Caddyfile:
```Caddyfile
# This is just an example Caddyfile content. Modify it to your liking.
example.com {
  reverse_proxy localhost:8080
  tls {
    dns cloudflare {env.CLOUDFLARE_API_TOKEN}
  }
}
```
- Change `example.com` to your domain/subdomain whose DNS `A` record is pointing to the public IP address of your VPS/server
- Change localhost to the `container_name` or IP of your service
- Change `8080` to the port at which your container is running on

## Next up
Manage your Caddyfile using [Upcaddy](https://github.com/deuts/upcaddy).
