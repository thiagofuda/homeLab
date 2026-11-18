# homeLab

Configurations and notes for a personal lab environment.

## Servers

### Dumbledore

VPS hosted on DigitalOcean.

Runs Ubuntu, Docker Compose, and Nginx, with HTTPS certificates from Let's Encrypt. Certificate renewal is handled by a systemd timer.

SSH access uses keys, and the server is protected by DigitalOcean Cloud Firewall.

Currently hosts [InfraLab](https://xdb.com.br).

- [InfraLab application code](https://github.com/thiagofuda/NetworkLab)
- [Dumbledore configuration and notes](servers/dumbledore/README.md)

### Fawkes

Directory reserved for another server or future experiments.

There is no active server under this name yet.

## Repository structure

```text
servers/
├── dumbledore/
│   ├── nginx/
│   ├── systemd/
│   └── README.md
└── fawkes/
```

Installation and configuration notes are in `servers/dumbledore/README.md`.

## Technologies

- Linux
- Docker and Docker Compose
- Nginx
- Let's Encrypt
- GitHub Actions
- Shell scripts

## Notes

This repository contains configurations and documentation for a personal lab environment.

Passwords, private keys, and other sensitive information should not be committed.

The structure may change as more servers, services, and experiments are added.
