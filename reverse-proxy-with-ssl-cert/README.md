<!-- omit in toc -->
# AutoMuteUs over SSL

By default, WebSocket communication between AmongUsCapture and Galactus is not encrypted.

This is an example of encrypting the communication by using a reverse proxy and a certificate issued by Let's Encrypt. Of course, the certificate will be renewed automatically on a regular basis.

![image](https://user-images.githubusercontent.com/2920259/109382377-6ded0c00-7923-11eb-8be2-68a89bd83dad.png)

This directory shows example files to use AutoMuteUs with reverse proxy in following methods:

- Use **Nginx** as the reverse proxy with the SSL certificate by **HTTP-01** challenge
- Use **Caddy** as the reverse proxy with the SSL certificate by **HTTP-01** or **TLS-ALPN-01** challenge
- Use **Caddy** as the reverse proxy with the SSL certificate by **DNS-01** challenge

<!-- omit in toc -->
## Table of Contents

- [Installation](#installation)
- [🚀 Use Nginx (HTTP-01 challenge)](#-use-nginx-http-01-challenge)
  - [Notes](#notes)
  - [Configuration](#configuration)
- [🚀 Use Caddy (HTTP-01 / TLS-ALPN-01 challenge)](#-use-caddy-http-01--tls-alpn-01-challenge)
  - [Notes](#notes-1)
  - [Configuration](#configuration-1)
- [🚀 Use Caddy (DNS-01 challenge)](#-use-caddy-dns-01-challenge)
  - [Notes](#notes-2)
  - [Configuration](#configuration-2)

## Installation

Choose one of the method you want to use and create your `.env` file using `sample.env` in appropriate directory. This `sample.env` contains additional common configuration and method-specific configuration.

- `REVERSEPROXY_EXTERNAL_PORT`
  - Speficy externally accessible port for the reverse proxy.
  - A reverse proxy will proxy `REVERSEPROXY_EXTERNAL_PORT` to the `GALACTUS_EXTERNAL_PORT` (ex. `8443` -> `8123`).
  - Make sure that `REVERSEPROXY_EXTERNAL_PORT` matches the port for the `GALACTUS_HOST`.
- `LETSENCRYPT_HOST`
  - Specify your own domain name.
  - The certificate will be issued to `LETSENCRYPT_HOST` by Let's Encrypt.
  - Make sure that `LETSENCRYPT_HOST` matches the host for the `GALACTUS_HOST`.
- `LETSENCRYPT_EMAIL`
  - Specify your email address to receive the renewal notice for the certificate from Let's Encrypt.
- `ACME_CA_URI`
  - Specify ACME API endpoint. Note that Let's Encrypt has [Rate Limit](https://letsencrypt.org/docs/rate-limits/) rules. It's recommendemd to use [Staging Server](https://letsencrypt.org/docs/staging-environment/) `https://acme-staging-v02.api.letsencrypt.org/directory` for testing or development purposes.
  - Staging servers have loose rate limits so it's easy to use, but the certificates issued by staging server are fake and cannot be used in practice. Once you are sure your reverse proxy works well and you are ready to issue a legitimate certificate, use the production server: `https://acme-v02.api.letsencrypt.org/directory`

The method-specific settings are described for each method at the rest of this page.

Once you have the `.env` file, you are ready to go.

```bash
docker compose up -d
```

## 🚀 Use Nginx (HTTP-01 challenge)

📁 [**nginx-http-01**](nginx-http-01)

### Notes

- Your host must be publicly reachable on both port `80/tcp` (fixed) and `8443/tcp` (modifiable via `REVERSEPROXY_EXTERNAL_PORT`).
  - The port `80/tcp` is used for `HTTP-01` challenge.
- The domains or subdomains you want to issue certificates for must correctly resolve to the host by public DNS server.

### Configuration

- `NGINXPROXY_TAG`, `ACME_COMPANION_TAG`
  - Refer [nginx-proxy](https://github.com/nginx-proxy/nginx-proxy/releases) and [acme-companion](https://github.com/nginx-proxy/acme-companion/releases), then specify the latest version without the leading `v`.

## 🚀 Use Caddy (HTTP-01 / TLS-ALPN-01 challenge)

📁 [**caddy-http-01**](caddy-http-01)

### Notes

- Your host must be publicly reachable on both port for ACME challenge and `8443/tcp` (modifiable via `REVERSEPROXY_EXTERNAL_PORT`).
  - The port `80/tcp` is used for `HTTP-01` challenge.
  - The port `443/tcp` is used for `TLS-ALPN-01` challenge.
- This configuration is forced to use `HTTP-01` challenge by default. If you want to force to use `TLS-ALPN-01` challenge, modify followings manually:
  - Change commenting / uncommenting the lines under `ports` for `caddy` service in `./docker-compose.yml`. Note that if you configured `443` as `REVERSEPROXY_EXTERNAL_PORT`, delete the line `- 443:443`, as it duplicates the same meaning under `ports`.
  - Change commenting / uncommenting the lines `disable_*_challenge` in `./caddy/Caddyfile`.
- The domains or subdomains you want to issue certificates for must correctly resolve to the host by public DNS server.
- The `docker-compose.yml` uses `./caddy/Caddyfile` as a relative path. Do not change the directory structure.

### Configuration

- `CADDY_TAG`
  - Refer [caddy](https://github.com/caddyserver/caddy/releases) and specify the latest version without the leading `v`.

## 🚀 Use Caddy (DNS-01 challenge)

📁 [**caddy-dns-01**](caddy-dns-01)

### Notes

- The advantage of this method is that the certificate can be issued even if the port `80/tcp` and `443/tcp` is blocked, but in return, it is a bit more complicated.
- Your host must be publicly reachable on the `8443/tcp` (modifiable via `REVERSEPROXY_EXTERNAL_PORT`). No other ports are required for the `DNS-01` challenge.
- These files are just a working example to use with Azure DNS. Depending on the DNS service which you are using, some of the files have to be modified to suit your environment. If you want to use other than Azure DNS, you will probably need to do the following:
  - Modify `Dockefile` to build your own Caddy image with appropriate DNS plugins.
  - Modify `Caddyfile` to use appropriate DNS plugins and to pass the required variables.
  - Modify `docker-compose.yml` and `.env` to pass the required environment variables.
- Check the [Caddy's wiki](https://caddy.community/t/how-to-use-dns-provider-modules-in-caddy-2/8148) for more information about using Caddy with the DNS provider.
- The `docker-compose.yml` uses `./caddy/Caddyfile` as a relative path. Do not change the directory structure.

### Configuration

- `CADDY_TAG`, `CADDY_DNS_TAG`
  - Refer [caddy](https://github.com/caddyserver/caddy/releases) and [caddy-dns-azure](https://github.com/caddy-dns/azure/tags), then specify the latest version without the leading `v`.
- `AZURE_*`
  - Specify your credentials and the related IDs/names for the Azure DNS.
