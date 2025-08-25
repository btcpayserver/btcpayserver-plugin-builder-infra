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