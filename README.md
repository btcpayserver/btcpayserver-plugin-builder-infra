# btcpayserver-plugin-builder-infra
The docker compose running the services for the BTCPay Server Plugin.

## How to use

Create a `.env` file with the following content:

```ini
PB_STORAGE_CONNECTION_STRING=<AZURE-STORAGE-CONNECTION-STRING>
PB_HOST=<DOMAIN-NAME>
```

Where you should replace:

* `<AZURE-STORAGE-CONNECTION-STRING>`: Replace with a connection string from a azure storage account. This is where the built plugins are hosted.
* `<DOMAIN-NAME>`: The domain name of your plugin builder website. HTTPS will be provisioned by let's encrypt automatically.

## Updating Services

If you need to update the version of a service (e.g., Plugin Builder), follow these steps:

1. Update the `docker-compose.yml` file in the repository and push commit.
2. On the server, run the following commands:

```bash
git fetch
git pull
docker compose up -d
```
