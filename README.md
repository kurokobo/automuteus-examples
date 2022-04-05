<!-- omit in toc -->
# Examples of AutoMuteUs deployments

This repository provides examples of how to use [Self-Hosted AutoMuteUs](https://github.com/denverquane/automuteus) in combination with other useful tools like Grafana dashboard, Let's Encrypt certficate, etc.

<!-- omit in toc -->
## Table of Contents

- [Contents](#contents)
  - [🚀 AutoMuteUs with Grafana Dashboard](#-automuteus-with-grafana-dashboard)
  - [🚀 AutoMuteUs over SSL](#-automuteus-over-ssl)
  - [🚀 Tips and Tricks](#-tips-and-tricks)
- [Contribution](#contribution)

## Contents

### 🚀 [AutoMuteUs with Grafana Dashboard](grafana-dashboard)

📁 [**grafana-dashboard**](grafana-dashboard)

This is an example of how to use AutoMuteUs with Grafana dashboard in easy way. All information will be aggregated in Prometheus automatically and will also be visualized in a pre-configured dashboard.

![image](https://user-images.githubusercontent.com/2920259/109378149-82290d00-7913-11eb-889e-83eb091d83e9.png)

### 🚀 [AutoMuteUs over SSL](reverse-proxy-with-ssl-cert)

📁 [**reverse-proxy-with-ssl-cert**](reverse-proxy-with-ssl-cert)

By default, WebSocket communication between AmongUsCapture and Galactus is not encrypted. This is an example of encrypting the communication by using a reverse proxy and a certificate issued by Let's Encrypt. Of course, the certificate will be renewed automatically on a regular basis.

![image](https://user-images.githubusercontent.com/2920259/109382377-6ded0c00-7923-11eb-8be2-68a89bd83dad.png)

### 🚀 Tips and Tricks

Some tips and tricks for someone in the future.

- 📋 [**Minimal bot permissions**](tips/minimal-bot-permissions.md)
  - This introduces the minimum privileges required for your bots on discord.
- 📋 [**Backup and Restore**](tips/backup-and-restore.md)
  - How to backup and restore your AutoMuteUs.
- 📋 [**AutoMuteUs on Raspberry Pi**](tips/raspberry-pi.md)
  - Some helpful information for getting AutoMuteUs to work on Raspberry Pi.

## Contribution

Contributions are welcome! If you have a good implementation, please feel free to create a new PR.
