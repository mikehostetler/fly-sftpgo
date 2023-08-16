# fly-sftpgo

This repo contains a template to run [SFTPGo](https://github.com/drakkan/sftpgo/) on [Fly.io](https://fly.io/) using the Apps V2 platform.  The SFTPGo machine scales to zero and wakes up upon access.

## Setup

You will need an available Postgres database to store your config.  You can set one up using the [Fly Postgres](https://fly.io/docs/postgres/).

*Fly Postgres Speedrun*
```
fly pg create
```

## Deploy

To host SFTPGo on Fly.io, clone this repo, enter the folder and run:

```
fly launch \
  --copy-config \
  --no-public-ips \
  --no-deploy \
  --ha=false

fly pg attach -a <YOUR_APP> <YOUR_PG_APP>

# Save the $DATABASE_URL from the output of the previous command

fly secrets set \
  SFTPGO_DATA_PROVIDER__DRIVER=postgresql \
  SFTPGO_DATA_PROVIDER__CONNECTION_STRING=<YOUR_CONNECTION_STRING>

fly secrets set SFTPGO_DATA_PROVIDER__DRIVER=postgresql SFTPGO_DATA_PROVIDER__CONNECTION_STRING=postgres://test_fly_sftpgo:oJVl7cRRuKsjkCV@twilight-dream-6791.flycast:5432/test_fly_sftpgo?sslmode=disable

fly deploy
```

## Configuration

### Service Configuration

Service configuration is best done through environment variables.  You can see the full range of available configuration options in the [SFTPGo Documentation](https://github.com/drakkan/sftpgo/blob/main/docs/full-configuration.md)

### User Configuration

To create users and stores within SFTPGo, log into the Web Admin panel. 

```
open https://<YOUR_APP>.fly.dev/web/admin
```

You can also proxy to the admin panel locally using the Fly CLI

```
fly proxy 8080
open https://localhost:8080/web/admin
```

### Run without Postgres

You can run SFTPGo without Postgres as the configuration store.  

1. Remove the `SFTPGO_DATA_PROVIDER__DRIVER` and `SFTPGO_DATA_PROVIDER__CONNECTION_STRING` environment variables from your app. 
2. Spin up SFTPGo and log into the web admin panel.  Set the configuration to your preferences.
3. Export the configuration as a JSON file
4. Add the following to your `fly.toml`

```
[env]
  SFTPGO_LOADDATA_FROM="/var/lib/sftpgo/sftpgo-backup.json" 
  SFTPGO_LOADDATA_CLEAN=0 
  SFTPGO_LOADDATA_MODE=1

# See https://community.fly.io/t/machine-files/14453 for other ways to specify this, as this backup file my contain sensitive information
[[files]]
  guest_path = "/var/lib/sftpgo/sftpgo-config-backup.json"
  local_path = "sftpgo-config-backup.json"
```

Your user configuration will now persist on reboots

## Feedback?

https://community.fly.io/ ...TODO

## Wishlist

I'd love to scale this out to a High Availability solution using SFPTGo clustering across multiple regions on Fly. If you have any ideas on how to do this, please let me know! PR's welcome!