# github.com/tiredofit/zerotier

[![GitHub release](https://img.shields.io/github/v/tag/tiredofit/docker-zerotier?style=flat-square)](https://github.com/tiredofit/docker-zerotier/releases)
[![Build Status](https://img.shields.io/github/workflow/status/tiredofit/docker-zerotier/build?style=flat-square)](https://github.com/tiredofit/docker-zerotier/actions?query=workflow%3Abuild)
[![Docker Stars](https://img.shields.io/docker/stars/tiredofit/zerotier.svg?style=flat-square&logo=docker)](https://hub.docker.com/r/tiredofit/zerotier/)
[![Docker Pulls](https://img.shields.io/docker/pulls/tiredofit/zerotier.svg?style=flat-square&logo=docker)](https://hub.docker.com/r/tiredofit/zerotier/)
[![Become a sponsor](https://img.shields.io/badge/sponsor-tiredofit-181717.svg?logo=github&style=flat-square)](https://github.com/sponsors/tiredofit)
[![Paypal Donate](https://img.shields.io/badge/donate-paypal-00457c.svg?logo=paypal&style=flat-square)](https://www.paypal.me/tiredofit)

## About

This will build a Docker Image for [Zerotier One](https://zerotier.com), A virtual ethernet switch client.

- Includes Zerotier One for setting up virtual private networks
- Also includes the management console [ZTNET](https://github.com/sinamics/next_ztnet)
- CoreDNS server included to fetch ip records from Zerotier Networks and serve under custom domains eg `custom.domain.local`
- Nginx as proxy to ZTNET for logging and authentication

## Maintainer

- [Dave Conroy](https://github.com/tiredofit/)

## Table of Contents

- [About](#about)
- [Maintainer](#maintainer)
- [Table of Contents](#table-of-contents)
- [Prerequisites and Assumptions](#prerequisites-and-assumptions)
- [Installation](#installation)
  - [Build from Source](#build-from-source)
  - [Prebuilt Images](#prebuilt-images)
    - [Multi Architecture](#multi-architecture)
- [Configuration](#configuration)
  - [Quick Start](#quick-start)
  - [Persistent Storage](#persistent-storage)
  - [Environment Variables](#environment-variables)
    - [Base Images used](#base-images-used)
    - [Container Options](#container-options)
    - [Controller Options](#controller-options)
    - [UI Options](#ui-options)
    - [Client Options](#client-options)
    - [DNS Options](#dns-options)
      - [Optional](#optional)
  - [Networking](#networking)
- [Maintenance](#maintenance)
  - [Shell Access](#shell-access)
- [Support](#support)
  - [Usage](#usage)
  - [Bugfixes](#bugfixes)
  - [Feature Requests](#feature-requests)
  - [Updates](#updates)
- [License](#license)
- [References](#references)

## Prerequisites and Assumptions
*  Assumes you are using some sort of SSL terminating reverse proxy such as:
   *  [Traefik](https://github.com/tiredofit/docker-traefik)
   *  [Nginx](https://github.com/jc21/nginx-proxy-manager)
   *  [Caddy](https://github.com/caddyserver/caddy)
*  Requires access to a  PostgreSQL Server if using the UI

## Installation
### Build from Source
Clone this repository and build the image with `docker build -t (imagename) .`

### Prebuilt Images
Builds of the image are available on [Docker Hub](https://hub.docker.com/r/tiredofit/zerotier).

```
docker pull tiredofit/zerotier:(imagetag)
```

Builds of the image are also available on the [Github Container Registry](https://github.com/tiredofit/zerotier/pkgs/container/zerotier)

```
docker pull ghcr.io/tiredofit/docker-zerotier:(imagetag)
```

The following image tags are available along with their tagged release based on what's written in the [Changelog](CHANGELOG.md):

| Container OS | Tag       |
| ------------ | --------- |
| Alpine       | `:latest` |

#### Multi Architecture
Images are built primarily for `amd64` architecture, and may also include builds for `arm/v7`, `arm64` and others. These variants are all unsupported. Consider [sponsoring](https://github.com/sponsors/tiredofit) my work so that I can work with various hardware. To see if this image supports multiple architecures, type `docker manifest (image):(tag)`

## Configuration

### Quick Start

* The quickest way to get started is using [docker-compose](https://docs.docker.com/compose/). See the examples folder for a working [compose.yml](examples/compose.yml) that can be modified for development or production use.

* Set various [environment variables](#environment-variables) to understand the capabilities of this image.
* Map [persistent storage](#data-volumes) for access to configuration and data files for backup.

### Persistent Storage

The following directories are used for configuration and can be mapped for persistent storage.

| Directory | Description                                                       |
| --------- | ----------------------------------------------------------------- |
| `/data/`  | volatile data in various subdirectories (controller, client, dns) |
| `/logs/`  | Logfiles                                                          |


* * *
### Environment Variables

#### Base Images used

This image relies on an [Alpine Linux](https://hub.docker.com/r/tiredofit/alpine) or [Debian Linux](https://hub.docker.com/r/tiredofit/debian) base image that relies on an [init system](https://github.com/just-containers/s6-overlay) for added capabilities. Outgoing SMTP capabilities are handlded via `msmtp`. Individual container performance monitoring is performed by [zabbix-agent](https://zabbix.org). Additional tools include: `bash`,`curl`,`less`,`logrotate`,`nano`.

Be sure to view the following repositories to understand all the customizable options:

| Image                                                  | Description                            |
| ------------------------------------------------------ | -------------------------------------- |
| [OS Base](https://github.com/tiredofit/docker-alpine/) | Customized Image based on Alpine Linux |
| [Nginx](https://github.com/tiredofit/docker-nginx/)    | Nginx webserver                        |


#### Container Options

| Variable   | Description                                                                 | Default         | `_FILE` |
| ---------- | --------------------------------------------------------------------------- | --------------- | ------- |
| `MODE`     | What mode `CLIENT` `CONTROLLER` `UI` `DNS` `STANDALONE` seperated by commas | `CONTROLLER,UI` |         |
| `LOG_PATH` | Where to store logs                                                         | `/logs/`        |         |


#### Controller Options

| Variable                              | Description                                                    | Default             | `_FILE` |
| ------------------------------------- | -------------------------------------------------------------- | ------------------- | ------- |
| `CONTROLLER_ALLOW_TCP_FALLBACK_RELAY` | Enable TCP relay                                               | `TRUE`              |         |
| `CONTROLLER_DATA_PATH`                | Zerotier volatile data                                         | `/data/controller/` |         |
| `CONTROLLER_ENABLE_METRICS`           | Enabler or disable prometheus metrics                          | `FALSE`             |         |
| `CONTROLLER_ENABLE_PORT_MAPPING`      | Enable Port mapping                                            | `TRUE`              |         |
| `CONTROLLER_LISTEN_PORT`              | Zerotier Controller listen port                                | `9993`              |         |
| `CONTROLLER_LOG_FILE`                 | Controller Log File                                            | `controller.log`    |         |
| `CONTROLLER_MANAGEMENT_NETWORKS`      | Comma seperated value of networks allowed to manage controller | `0.0.0.0/0`         |         |
| `CONTROLLER_USER`                     | What username to run controller as                             | `root`              |         |
| `CONTROLLER_NETWORK`                  | (optional) Networks to join as Controller                      |                     | x       |
| `CONTROLLER_IDENTITY_PRIVATE`         | (optional) Pre generated private identity                      |                     | x       |
| `CONTROLLER_IDENTITY_PUBLIC`          | (optional) Pre generated public identity                       |                     | x       |

#### UI Options

| Variable            | Description                                        | Default                                      | `_FILE` |
| ------------------- | -------------------------------------------------- | -------------------------------------------- | ------- |
| `ENABLE_NGINX`      | If wanting to use Nginx as proxy to UI_LISTEN_PORT | `TRUE`                                       |         |
| `NGINX_LISTEN_PORT` | Nginx Listening Port                               | `80`                                         |         |
| `UI_CONTROLLER_URL` | How can the UI access the controller               | `http://localhost:${CONTROLLER_LISTEN_PORT}` |         |
| `UI_DB_HOST`        | DB Host for Postgresql                             |                                              |         |
| `UI_DB_NAME`        | DB Name for UI                                     |                                              |         |
| `UI_DB_PASS`        | Password for UI_DB_USER                            |                                              |         |
| `UI_DB_PORT`        | DB Port for Postgresql                             | `5432`                                       |         |
| `UI_DB_USER`        | DB User for UI_DB_NAME                             |                                              |         |
| `UI_LISTEN_PORT`    | What port for the UI to listen on                  | `3000`                                       |         |
| `UI_SECRET`         | Random secret for session and cookie storage       | `random`                                     |         |
| `UI_SITE_NAME`      | Site name to display on UI                         | `ZTNET`                                      |         |


#### Client Options

| Variable                          | Description                               | Default         | `_FILE` |
| --------------------------------- | ----------------------------------------- | --------------- | ------- |
| `CLIENT_ALLOW_TCP_FALLBACK_RELAY` | Enable TCP relay                          | `TRUE`          |         |
| `CLIENT_DATA_PATH`                | Zerotier volatile data                    | `/data/client/` |         |
| `CLIENT_ENABLE_METRICS`           | Enabler or disable prometheus metrics     | `FALSE`         |         |
| `CLIENT_ENABLE_PORT_MAPPING`      | Enable Port mapping                       | `TRUE`          |         |
| `CLIENT_LISTEN_PORT`              | Zerotier client listen port               | `9995`          |         |
| `CLIENT_LOG_FILE`                 | Client Log File                           | `client.log`    |         |
| `CLIENT_USER`                     | What username to run controller as        | `root`          |         |
| `CLIENT_NETWORK`                  | (optional) Networks to join as client     |                 | x       |
| `CLIENT_IDENTITY_PRIVATE`         | (optional) Pre generated private identity |                 | x       |
| `CLIENT_IDENTITY_PUBLIC`          | (optional) Pre generated public identity  |                 | x       |

#### DNS Options
| Variable          | Description                                                                                         | Default                 | `_FILE` |
| ----------------- | --------------------------------------------------------------------------------------------------- | ----------------------- | ------- |
| `ZTNET_API_HOST`  | API Hostname of ZTNET Api Server                                                                    | `http://localhost:3000` |
| `ZTNET_API_TOKEN` | API Token able to fetch information from ZTNET Server                                               |                         |
| `ZT_NETWORKS`     | Networks as org:dnsname:network (multiple networks separated by comma) eg org123:example.com:net123 |                         |         |

##### Optional
| Variable                    | Description                                                  | Default                                  | `_FILE` |
| --------------------------- | ------------------------------------------------------------ | ---------------------------------------- | ------- |
| `DNS_ALLOW_AXFR`            | Allow Zone Transfers to secondary servers                    | `true`                                   |         |
| `DNS_AXFR_HOSTS`            | Hosts to notify when files has changed, or allow transfer to | `*`                                      |         |
| `DNS_BIND_ALL`              | Bind CoreDNS to all interfaces                               | `true`                                   |         |
| `DNS_BIND_LOCALHOST`        | Bind CoreDNS to localhost                                    | `true`                                   |         |
| `DNS_BIND_ZEROTIER`         | Bind CoreDNS to ZeroTier interfaces                          | `true`                                   |         |
| `DNS_BIND_IP`               | Bind CoreDNS to specific IP addresses                        |                                          |         |
| `DNS_CONFIG_FILE`           | Path to CoreDNS configuration file                           | `/etc/coredns/Corefile`                  |         |
| `DNS_CUSTOM_DATA_PATH`      | Path to CoreDNS hosts                                        | `/data/dns/`                             |         |
| `DNS_CUSTOM_HOST[00,01]`    | DNS Custom host entry eg `127.0.0.1 hostname.domain`         |                                          |         |
| `DNS_ENABLE_FORWARD`        | Enable CoreDNS forwarding                                    | `true`                                   |         |
| `DNS_FORWARD_MODE`          | CoreDNS forward mode (`system` or `upstream`)                | `system`                                 |         |
| `DNS_FORWARD_UPSTREAM_HOST` | CoreDNS forward upstream hosts                               | `dns://1.1.1.1:53 dns://1.0.0.1:53`      |         |
| `DNS_INCLUDE_CONFIG`        | Path to to custom CoreDNS Configuration to include           | `${DNS_CUSTOM_DATA_PATH}/config.include` |         |
| `DNS_INCLUDE_HOSTS`         | Path to to custom Hosts file to include                      | `${DNS_CUSTOM_DATA_PATH}/hosts.include`  |         |
| `DNS_LISTEN_PORT`           | CoreDNS listen port                                          | `53`                                     |         |


### Networking

| Port   | Protocol | Description          |
| ------ | -------- | -------------------- |
| `53`   | `udp`    | CoreDNS              |
| `80`   | `tcp`    | Nginx                |
| `3000` | `tcp`    | ZTNET web UI         |
| `9993` | `udp`    | Zerotier Control API |
| `9994` | `udp`    | Zerotier Client API  |


## Maintenance
### Shell Access

For debugging and maintenance purposes you may want access the containers shell.

```bash
docker exec -it (whatever your container name is) bash
```
## Support

These images were built to serve a specific need in a production environment and gradually have had more functionality added based on requests from the community.
### Usage
- The [Discussions board](../../discussions) is a great place for working with the community on tips and tricks of using this image.
- [Sponsor me](https://tiredofit.ca/sponsor) for personalized support.

### Bugfixes
- Please, submit a [Bug Report](issues/new) if something isn't working as expected. I'll do my best to issue a fix in short order.

### Feature Requests
- Feel free to submit a feature request, however there is no guarantee that it will be added, or at what timeline.
- [Sponsor me](https://tiredofit.ca/sponsor) regarding development of features.

### Updates
- Best effort to track upstream changes, More priority if I am actively using the image in a production environment.
- [Sponsor me](https://tiredofit.ca/sponsor) for up to date releases.

## License
MIT. See [LICENSE](LICENSE) for more details.

## References

* <https://zerotier.com>
