# homeLab

This is my personal infrastructure lab.

I work with IT infrastructure, Linux, and cloud platforms, and I created this
repository to document my own servers, configurations, experiments, and things
I learn in my free time.

My goal is to have a place where I can test ideas, build small projects, and
practice infrastructure and automation without making things more complicated
than they need to be.

## Servers

### Dumbledore

Dumbledore is my active server, hosted on DigitalOcean. It runs Ubuntu 26.04 LTS
with Docker and Docker Compose. Nginx acts as the reverse proxy, HTTPS uses
Let's Encrypt, and a systemd timer checks for certificate renewal. SSH access
uses keys, and a DigitalOcean Cloud Firewall protects the server.

Dumbledore currently hosts InfraLab. Its application code is in the
[NetworkLab repository](https://github.com/thiagofuda/NetworkLab); this
repository contains the server configuration.

### Fawkes

The `servers/fawkes/` directory is reserved for future experiments. It does not
currently contain an active server configuration.

## Repository Structure

```text
homeLab/
├── docs/                         # Currently empty
├── servers/
│   ├── dumbledore/
│   │   ├── docker/               # Currently empty
│   │   ├── nginx/                # Reverse proxy configuration
│   │   ├── systemd/              # Certificate renewal timer and service
│   │   └── README.md             # Dumbledore deployment notes
│   └── fawkes/                   # Reserved; currently empty
└── worflow/
    └── validation.yml
```

## Current Projects

- [InfraLab](https://xdb.com.br): public tools for infrastructure and network
  diagnostics.
- [Application source](https://github.com/thiagofuda/NetworkLab): Python and
  FastAPI code for InfraLab.
- [Infrastructure configuration](https://github.com/thiagofuda/homeLab): this
  repository.

## What I Want to Explore

- Linux server administration.
- Infrastructure as Code.
- Cloud platforms.
- Docker and containers.
- CI/CD and automation.
- Monitoring and observability.
- Python for infrastructure automation.
- AI-assisted infrastructure diagnostics.

## Notes

This is a personal lab, and it will change as I learn and try different
approaches.

I use GitHub Copilot and ChatGPT to help review configurations, discuss ideas,
and understand new technologies.

I try to keep everything simple, documented, and easy to maintain.

The most important part is to learn something new and have fun along the way.