# balena-nextcloud

nextcloud stack for balenaCloud

## Requirements

- RaspberryPi3, RaspberryPi4, or a similar device supported by BalenaCloud
- Custom domain name with DNS pointing to your balena device (eg. nextcloud.your-domain.com)
- (optional) A USB storage device with a partition labeled `NEXTCLOUD`

## Getting Started

To get started you'll first need to sign up for a free balenaCloud account and flash your device.

<https://www.balena.io/docs/learn/getting-started>

## Deployment

Once your account is set up, deployment is carried out by downloading the project and pushing it to your device either via Git or the balenaCLI.

### Application Environment Variables

Application envionment variables apply to all services within the application, and can be applied fleet-wide to apply to multiple devices.

|Name|Example|Purpose|
|---|---|---|
|`NEXTCLOUD_TRUSTED_DOMAINS`|`nextcloud.your-domain.com`|space-separated list of trusted domains for remote access|
|`MYSQL_ROOT_PASSWORD`|`********`|password that will be set for the MariaDB root account|
|`MYSQL_PASSWORD`|`********`|password that will be set for the MariaDB nextcloud account|
|`TRAEFIK_PROVIDERS_DOCKER_DEFAULTRULE`|``Host(`nextcloud.your-domain.com`)``|provide your custom domain here|
|`TRAEFIK_CERTIFICATESRESOLVERS_TLSCHALLENGE_ACME_EMAIL`|`foo@bar.com`|email address to use for ACME registration|
|`TRAEFIK_CERTIFICATESRESOLVERS_TLSCHALLENGE_ACME_CASERVER`|`https://acme-staging-v02.api.letsencrypt.org/directory`|(optional) specify a different CA server to use|
|`TZ`|`America/Toronto`|(optional) inform services of the [timezone](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) in your location|

## Usage

### prepare external storage for nextcloud

Connect to the `Host OS` Terminal and run the following:

```bash
# o - clear the in memory partition table
# n - new partition
# p - primary partition
# 1 - partition number 1
# default - start at beginning of disk
# default - extend partition to end of disk
# w - write the partition table
printf "o\nn\np\n1\n\n\nw\n" | fdisk /dev/sda
mkfs.ext4 /dev/sda1 -L NEXTCLOUD
```

Restart the `nextcloud` service and the storage should be mounted at `/files`.

## fix nextcloud overview warnings

Connect to the `nextcloud` terminal and run the following:

```bash
apt-get update && apt-get install sudo

# fix nextcloud reverse proxy warnings
sudo -u www-data php ./occ config:system:set trusted_proxies 1 --value='traefik'
sudo -u www-data php ./occ config:system:set overwrite.cli.url --value='https://nextcloud.your-domain.com/'
sudo -u www-data php ./occ config:system:set overwritehost --value='nextcloud.your-domain.com'
sudo -u www-data php ./occ config:system:set overwriteprotocol --value='https'

# fix nextcloud database warnings
sudo -u www-data php ./occ db:add-missing-indices
sudo -u www-data php ./occ db:convert-filecache-bigint
```

Now the expected warnings in Settings->Overview should be gone.

## Contributing

Please open an issue or submit a pull request with any features, fixes, or changes.

## Author

Kyle Harding <https://klutchell.dev>

## Acknowledgments

- <https://hub.docker.com/_/nextcloud/>
- <https://hub.docker.com/_/mariadb/>
- <https://hub.docker.com/_/traefik/>

## License

[MIT License](./LICENSE)
