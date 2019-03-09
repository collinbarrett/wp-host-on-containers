[![Build status](https://dev.azure.com/collinbarrett/wp-host-on-containers/_apis/build/status/wp-host-on-containers-CI)](https://dev.azure.com/collinbarrett/wp-host-on-containers/_build/latest?definitionId=4)

# wp-host-on-containers

Continuously deploying WordPress sites in docker containers to a [$5/mo DigitalOcean Ubuntu VPS](https://m.do.co/c/fea63c0a77d1 "DigitalOcean Affiliate Link") via Azure Pipelines

A modern, containerized `v2` of [wp-vps-build-guide](https://github.com/collinbarrett/wp-vps-build-guide).

# The Stack

- GitHub
- Azure Pipelines
- DigitalOcean
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

1. Create a minimum-size droplet with the latest Ubuntu LTS.
    - Enable backups.
    - Enable IPv6 and Monitoring.
    - Include a pre-configured SSH key.
2. Follow the DigitalOcean guide for [Initial Server Setup with Ubuntu](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04).
3. Set `PermitRootLogin` to `no` in `/etc/ssh/sshd_config`.
4. `sudo apt-get update; sudo apt-get upgrade; sudo apt-get dist-upgrade; sudo apt-get autoremove; sudo reboot now`
5. `sudo apt-get install fail2ban`
6. Enable [Ubuntu automatic updates](https://help.ubuntu.com/lts/serverguide/automatic-updates.html.en).

## Register with Azure DevOps

TBD

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
4. `sudo chown root:root ~/cert`

# Azure Pipelines Setup

TBD