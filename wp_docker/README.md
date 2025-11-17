# WordPress Stack

Docker Compose bundles WordPress, MySQL, and phpMyAdmin for local development or a small single-host deployment. The compose file lives in this folder and already references the correct volumes, networks, and health checks.

## Prerequisites

- Docker Engine 24+
- Docker Compose plugin (`docker compose` command)

## Configure Secrets

Edit `docker-compose.yml` before the first run and replace `change_me_root` and `change_me_app` with strong passwords. For production, pass secrets through an `.env` file or your orchestrator instead of committing them.

## Run the Stack

```bash
cd wp_docker
docker compose up -d
```

Services:

- WordPress (Apache/PHP 8.2) on <http://localhost:8000>
- phpMyAdmin on <http://localhost:8081>
- MySQL 8.0 exposed only inside the `wpsite` network

### Shutdown and Cleanup

```bash
docker compose down --volumes
```

## Nginx Reverse Proxy

The host-level Nginx configuration is in `nginx/wordpress.conf`. Update the `server_name` and certificate paths so they match your domain before deploying. The file already:

- redirects HTTP to HTTPS and serves ACME challenges from `/var/www/certbot`
- proxies requests to the WordPress container published on port `8000`
- forwards the correct `X-Forwarded-*` headers so WordPress can detect HTTPS

### Setup the Nginx Config

1. Copy the sample file into your Nginx config directory (usually `/etc/nginx/conf.d`). Run these commands from the repo root:

   ```bash
   sudo mkdir -p /etc/nginx/conf.d
   sudo cp nginx/wordpress.conf /etc/nginx/conf.d/wordpress.conf
   ```

2. Edit the copied file:

   - replace `server_name` with the public hostname(s)
   - point `ssl_certificate` and `ssl_certificate_key` at the certificates issued for that host
   - adjust the `proxy_pass http://127.0.0.1:8000;` line if the WordPress container is published on a different port

3. Ensure the ACME webroot exists (`sudo mkdir -p /var/www/certbot`) before running Certbot so HTTP-01 challenges succeed.

4. Test and reload Nginx once the file is in place:

   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   ```

5. Obtain or renew certificates with Certbot using the same `server_name` values from the config.
