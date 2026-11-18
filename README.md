# btcpayserver-plugin-builder-infra
The docker compose running the services for the BTCPay Server Plugin.

## First deployment

Use a Linux/amd64 Docker host with gVisor installed, then prepare the runtime and
the private build directory:

```bash
sudo runsc install
sudo systemctl restart docker
docker info --format '{{json .Runtimes}}'
sudo install -d -m 0700 /mnt/pluginbuilder-build-scratch
```

The runtime list must contain `runsc`. The scratch directory must contain only
disposable build data and must not be a symlink or filesystem root.

Keep Docker's network firewall rules enabled. The broker creates an internal
worker network and a Squid proxy with a restricted destination policy.
Custom host firewall rules are optional defense in depth, not a prerequisite
for deployment: they can limit network access if the proxy itself is compromised.
Validate network isolation and a complete build on the VM before opening builds
to users.

Create the shared broker token outside the repository:

```bash
sudo install -d -m 0700 /etc/pluginbuilder/secrets
sudo bash -c '
  umask 077
  set -o noclobber
  openssl rand -hex 32 > /etc/pluginbuilder/secrets/build-broker-token
'
```

Create a private `.env` file:

```ini
PB_STORAGE_CONNECTION_STRING=<AZURE-STORAGE-CONNECTION-STRING>
PB_HOST=<DOMAIN-NAME>
PB_BUILD_SCRATCH_ROOT=/mnt/pluginbuilder-build-scratch
PB_BUILD_BROKER_TOKEN_FILE=/etc/pluginbuilder/secrets/build-broker-token
```

`PB_BUILD_BROKER_TOKEN_FILE` is the absolute path, not the token value.

Validate the configuration and start the deployment:

```bash
docker compose config --quiet
docker compose up -d --wait --wait-timeout 1200
docker compose ps
```

## Updates

Release the application, broker and worker with the same version and update
all three tags in `docker-compose.yml`. CI checks that their release tags match.
The broker pulls its pinned upstream Squid proxy image itself.
Drain builds before deploying:

```bash
git pull
docker compose config --quiet
docker compose up -d --wait --wait-timeout 1200
```

### Nginx template updates

A change to the bind-mounted `nginx.tmpl` does not recreate `nginx-gen` during
`docker compose up`. After the update commands above, restart the generator to
render the updated template. The generator also signals Nginx to reload:

```bash
docker compose restart nginx-gen
docker compose logs --since 2m nginx-gen
docker compose exec nginx nginx -t
docker compose logs --since 2m nginx
```

Confirm successful template generation and reload in the logs. If generation or
validation fails, the mitigation is not verified: fix the error and repeat the
steps. Inspect `docker compose exec nginx nginx -T` on the host to verify the
effective configuration; do not share its output if it contains private settings.

Nginx is pinned to 1.31.9, which fixes
[CVE-2026-123007 and CVE-2026-123633](https://nginx.org/en/security_advisories.html).
HTTP/2 is also disabled on every HTTPS listener; clients use HTTP/1.1.
Verify that the effective configuration has no `http2` listener parameter or
`http2 on` directive, including custom vhost configuration. This mitigation does
not replace running a fixed Nginx image (1.30.7+ or 1.31.9+). After changing
the image tag, pull the image and recreate the Nginx container; a configuration
reload alone cannot update its binary.
