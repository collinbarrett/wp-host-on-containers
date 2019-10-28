|                                                     | Build                                                                                                                                                                                                        | Release                                                                                                                         |
| --------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| nginx                                               | [![Build status](https://dev.azure.com/collinbarrett/wp-host-on-containers/_apis/build/status/nginx)](https://dev.azure.com/collinbarrett/wp-host-on-containers/_build/latest?definitionId=22)               | ![Release status](https://vsrm.dev.azure.com/collinbarrett/_apis/public/Release/badge/5afed59e-e8b7-4bd3-9704-5c9d945dffd4/2/2) |
| [collinmbarrett.com](https://collinmbarrett.com/)   | [![Build status](https://dev.azure.com/collinbarrett/wp-host-on-containers/_apis/build/status/collinmbarrett)](https://dev.azure.com/collinbarrett/wp-host-on-containers/_build/latest?definitionId=9)       | ![Release status](https://vsrm.dev.azure.com/collinbarrett/_apis/public/Release/badge/5afed59e-e8b7-4bd3-9704-5c9d945dffd4/4/4) |
| [jennythebaker.com](https://jennythebaker.com/)     | [![Build status](https://dev.azure.com/collinbarrett/wp-host-on-containers/_apis/build/status/jennythebaker-com)](https://dev.azure.com/collinbarrett/wp-host-on-containers/_build/latest?definitionId=10)   | ![Release status](https://vsrm.dev.azure.com/collinbarrett/_apis/public/Release/badge/5afed59e-e8b7-4bd3-9704-5c9d945dffd4/5/5) |
| [sycamorememphis.org](https://sycamorememphis.org/) | [![Build status](https://dev.azure.com/collinbarrett/wp-host-on-containers/_apis/build/status/sycamorememphis-org)](https://dev.azure.com/collinbarrett/wp-host-on-containers/_build/latest?definitionId=11) | ![Release status](https://vsrm.dev.azure.com/collinbarrett/_apis/public/Release/badge/5afed59e-e8b7-4bd3-9704-5c9d945dffd4/6/6) |

# wp-host-on-containers

Continuously deploying WordPress sites in docker containers to a [\$5/mo DigitalOcean Ubuntu VPS](https://m.do.co/c/fea63c0a77d1 "DigitalOcean Affiliate Link") via Azure Pipelines

A modern, containerized `v2` of [wp-vps-build-guide](https://github.com/collinbarrett/wp-vps-build-guide).

Note: This repo is primarily for my personal DevOps process and not meant to be directly re-usable by anyone. However, it could potentially serve as a reference for anyone trying to do something similar.

# The Stack

- GitHub
- Azure Pipelines
- [DigitalOcean](https://m.do.co/c/fea63c0a77d1 "DigitalOcean Affiliate Link")
- Ubuntu LTS
- Docker CE
- Docker Compose
- MariaDB
- php-fpm
- WordPress
- nginx
- Cloudflare

# Deployment Target Setup

## Initial Setup

1. Create a minimum-size Standard Droplet with the latest Ubuntu LTS.
   - Add backups.
   - Enable Monitoring.
   - Include a pre-configured SSH key.
2. Follow the DigitalOcean guide for [Initial Server Setup with Ubuntu](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04).
3. Set `PermitRootLogin` to `no` in `/etc/ssh/sshd_config`.
4. `sudo apt install fail2ban`
5. Enable [Ubuntu automatic updates](https://help.ubuntu.com/lts/serverguide/automatic-updates.html.en).

## Register with Azure DevOps

1. Visit `https://dev.azure.com/<ORGANIZATIONNAME>/_settings/deploymentpools`.
2. Add a new deployment pool.
3. Execute the provided installation script for linux.

## Install Docker

1. See the [official docs](https://docs.docker.com/install/linux/docker-ce/ubuntu/) for installing on Ubuntu.
2. Complete the desired [linux postinstall procedures](https://docs.docker.com/install/linux/linux-postinstall/).

## Install Docker Compose

See the [official docs](https://docs.docker.com/compose/install/) for installing on linux.

## TLS Certificates

I use [Cloudflare's free ssl certificates](https://www.cloudflare.com/ssl/).

1. SFTP the following to `~/cert`:
   - domain-specific key `.pem` files
   - domain-specific cert `.pem` files
   - [Cloudflare Authenticated Origin Pull cert](https://support.cloudflare.com/hc/en-us/article_attachments/201243967/origin-pull-ca.pem)
2. `sudo openssl dhparam -out ~/cert/dhparam.pem 2048`
3. `sudo chmod -R 600 ~/cert`
4. `sudo chown -R root:root ~/cert`

# Azure Pipelines Setup

TBD

# Additional Configuration After First Release

## Restrict MariaDB User Permissions

Since the default user initialized by the MariaDB docker container is granted all privileges on the default database, we want to restrict that to just the permissions required by normal WordPress operations. Replace angle brackets with our actual values.

- `docker exec -i -t <MariaDB_container_name> /bin/bash`
- `mysql -u root -p`
- `REVOKE ALL PRIVILEGES ON <_WORDPRESS_DB_NAME>.* FROM '<_WORDPRESS_DB_USER>'@'%';`
- `GRANT SELECT, INSERT, UPDATE, DELETE ON <_WORDPRESS_DB_NAME>.* TO '<_WORDPRESS_DB_USER>'@'%';`

## WordPress Auto-Updates

For now, enable auto-updates using [WordPress's built-in functionality](https://codex.wordpress.org/Configuring_Automatic_Background_Updates). I plan to extend the flexibility of this in the future by using wp-cli triggered from cron jobs to perform udpates and maintenance.

- `docker exec -i -t <WordPress_container_name> /bin/bash`
- `apk add nano` (Or other text editor of choice. Since I have `--force-recreate` on releases, this will get removed during the next release to keep the container slim.)

### WordPress Core

- Add `define( 'WP_AUTO_UPDATE_CORE', true );` to `wp-config.php`

### Plugins

- Add `add_filter( 'auto_update_plugin', '__return_true' );` to theme's `functions.php`

# TODO List

- [x] Use secrets for database configurations.
- [x] Limit permissions of WordPress database user.
- [ ] Implement scheduled backups of databases and files.
  - [x] Phase 1: using DigitalOcean's droplet backups
  - [ ] Phase 2: docker-compose named volume backups
- [ ] Implement auto-updates of WordPress core and plugins.
  - [x] Phase 1: using WordPress's auto-updater
  - [ ] Phase 2: scheduled using wp-cli
- [ ] Implement scheduled [wp-sweep](https://github.com/lesterchan/wp-sweep) via [wp-cli](https://wp-cli.org/).
- [x] Implement miscellaneous nginx best practices for speed and security.
- [x] Implement redis object caching.
- [ ] Implement fastCGI page caching.
- [ ] Tune MariaDB instances for WordPress performance.
