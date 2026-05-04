# Homelab

This repo is my attempt to finally document my homelab in a more organized way.

I’ve been building pieces of this environment over the last few years — sometimes to learn something specific, sometimes to solve a real problem at home, and sometimes just because I wanted to see if I could get something working. At some point I realized that the actual value was not just in having the services running, but in being able to explain what I built, why I built it, how it works, and what I learned along the way.

So this repository is a bit of a “going back and writing it down” project.

The goal is not to present a perfect enterprise environment. This is a real homelab, which means some things were built cleanly, some things were figured out through trial and error, and a few things probably changed three times before landing where they are now. I want this repo to capture that process honestly: the setup, the decisions, the mistakes, the fixes, and the improvements over time.

---

## Current Status

This documentation is being rebuilt after the fact. Some services were already running before this repository existed, so part of the work here is reconstructing the decisions, setup steps, troubleshooting, and lessons learned from the current environment.

Some details are intentionally generalized or removed before publishing, including usernames, private hostnames, IP addresses, credentials, tokens, API keys, family information, and screenshots that expose account-specific details.

---

## How This Started

This homelab has been a journey, to say the least.

It originally started with the purchase of a Synology DS1515+ for media storage and backup. At that stage, the goal was pretty straightforward: have a reliable central place for media, photos, documents, and household backups. The Synology filled that role well by hosting media through Plex and acting as a storage/backup target for important files.

For a while, the NAS also had cloud backup enabled through CrashPlan running in a Docker container. That worked as an off-site backup option until CrashPlan shifted toward a per-terabyte pricing model, which no longer made sense financially for the amount of data being stored. That change pushed me to rethink backup strategy and become more intentional about what should be stored locally, what should be backed up externally, and what services were worth maintaining long-term.

A few years later, the homelab started expanding beyond basic storage and media. I became interested in running additional systems myself, starting with hosting a server for a few friends. That opened the door to learning more about Docker, basic SQL, database management, dependencies, service configuration, and the kind of troubleshooting that comes with keeping something running for other people.

Over time, the environment kept growing as my learning expanded and the needs at home became more specific. As I learned more about systems administration and infrastructure, I started adding monitoring tools so I could keep a better eye on system health instead of only reacting when something broke.

Networking also became a bigger focus. My router was converted to DD-WRT so I could get better access to the networking side of the environment and maintain more control over the home network. Pi-hole was added next to take control of DNS, provide ad blocking, reduce unwanted traffic, add some protection against known unwanted or malicious domains, and give me better visibility into what was moving in and out of the network. As the network grew, using Pi-hole for DHCP also became valuable because it gave me more centralized control over client addressing and DNS behavior.

Immich grew out of a similar desire for more control, but focused specifically on photos and videos. The question became: why leave all of our family images and videos sitting only on someone else’s server if I had the ability to bring that data back under local control? As AI training on public and cloud-hosted images and videos became a larger concern, Immich became my first serious step toward shifting more of our media storage and backup workflow back into the homelab.

That created new challenges. Compute was one of the biggest ones. I quickly found that the Synology, while excellent as a NAS and storage platform, was not really designed to handle every service I wanted to run at the same time. That is where the Dell Micro came into the stack. Adding it gave me a separate Linux-based compute host where I could offload Docker services from the Synology and separate active application workloads from the storage system.

Portainer was added as the environment became more complex and I needed a cleaner way to manage containers across systems. That also created its own learning curve, because I had to think more carefully about how containers were deployed, where they should run, how ports were mapped, how volumes were handled, and how to keep the setup understandable as it grew.

Over time, I also began shifting some Docker services from standalone Docker Compose files into Portainer Stacks. The goal was not to make the environment more complicated, but to make it easier to see, manage, update, and control from one place. This also helped me treat the services more like managed infrastructure instead of a loose collection of containers that happened to be running.

Tailscale was added so I could provide outside access to devices and services without relying on traditional port forwarding or exposing everything through a public proxy. This became especially useful for Immich because it allowed devices to access the photo backup system both at home and remotely while still keeping the overall access model private.

Backups also continued to evolve. AWS S3 backup was added to the Synology as an additional layer for photo and video media. The backup data lands in an S3 bucket and then lifecycle rules move it into Glacier within 24 hours. Since most of this media is incremental, archival, and not constantly edited after staging, Glacier made sense as a low-cost long-term backup option. This brought the cost down to roughly $12/month, which was far lower than many other cloud backup options I had explored.

Time Machine backups were also enabled through the Synology, using a dedicated WD hard drive connected over USB. This is not a fancy setup, but it provides a simple household backup target that can support weekly incremental backups for family devices.

Some services are still experimental. Wyze Bridge has been tested as a way to expose Wyze security cameras over RTSP, but the setup has been unreliable and finicky. Long term, this will likely be replaced with a PoE camera system that can integrate more cleanly with Frigate NVR.

Home Assistant is also running, but it is not fully implemented across the house yet. The service works, but broader household adoption would require more time, device integration, and family onboarding.

What started as a NAS for media and backup slowly turned into a practical learning environment for systems administration, networking, storage, monitoring, remote access, Docker management, backups, and self-hosted services.

---

## High-Level Goals

The main goals of this homelab are:

* Build and maintain real services that solve actual home and family needs.
* Practice Linux, Docker, networking, storage, monitoring, and remote access in a hands-on environment.
* Learn through troubleshooting instead of only reading documentation or studying for certifications.
* Create repeatable documentation that explains not just what is running, but why it was built that way.
* Use the environment as a practical portfolio of IT, systems administration, networking, and security-adjacent work.

---

## Design Philosophy

A lot of the design choices in this homelab come down to a few practical rules: keep storage reliable, keep active services responsive, avoid unnecessary public exposure, and make the system usable for the household without adding friction.

### Synology vs. Dell Micro

The Synology is primarily storage-focused. It is a solid NAS, but with a 4-core CPU, it is not the place I want to run every active workload. I use it for the more storage-heavy and distributed side of the environment: media storage, backups, photo/document storage, USB-attached backup drives, and services that make sense to keep close to the data.

Plex stayed on the Synology because it was already heavily used and moving it was proving messier than it was worth at the time. Since it was actively serving the household, I chose stability over unnecessary migration.

The Dell Micro is the main Docker and service host. It has significantly more compute available, with 14 cores, and is better suited for running active containers, monitoring tools, logging, local AI services, and workloads that benefit from a standard Linux host. The tradeoff is storage: it has a fast but much smaller 500 GB SSD, so I treat it more as a compute node than a long-term storage system.

In practice:

* Synology = storage, backups, media/archive workflows, NAS-first services, USB-attached backup drives
* Dell Micro = Docker services, monitoring, logs, DNS-related tooling, local AI services, and Linux-first workloads

### Access and Security

My default approach is to keep systems LAN-only whenever possible. I try to avoid public exposure unless there is a clear reason for it, and I prefer to keep services siloed rather than making everything broadly reachable.

The household uses many of these services even if they do not know they are using them, so reliability and low disruption matter. A technically interesting setup is not useful if it constantly breaks things for everyone else.

Tailscale is the main remote-access method because it gives me a way to reach internal services without opening a large number of ports or exposing services directly to the public internet. The goal is limited remote access, minimal open ports, and controlled access to only what needs to be reachable.

### Backup Philosophy

My backup approach is local-first, with off-site/archive backup added where it makes sense.

Local storage is important because the household depends on the environment being available without thinking about it. The goal is to provide the best service possible with minimal disruption, because friction makes people less likely to actually use the system.

For disaster recovery and long-term retention, I use external/off-site options where appropriate. AWS S3 with Glacier lifecycle rules is used for long-term archival photo and video backup. Time Machine backups are kept simple by using the Synology with a dedicated USB-attached WD drive so household devices have an easy backup target.

### Testing and Learning

The lab also includes a few Linux and Windows 10/11 virtual machines for general testing. These VMs give me a safe place to test operating systems, tools, troubleshooting steps, and configurations before touching anything more important.

One of the main purposes of this repo is to show that I am constantly testing, building, and working with new systems. My day-to-day technical work is not always heavily server-focused, but I use these tools and skills regularly in my own environment. I test systems before incorporating them, document what I learn, and keep evolving the setup as my needs and knowledge grow.

---

## Environment Overview

The homelab is built around two main systems:

### Dell Micro Ubuntu Server

The Dell Micro acts as the main Docker and monitoring host. It runs most of the active services that need to be available on the local network, including DNS-related tooling, monitoring, logging, photo backup services, and local AI tools.

Primary responsibilities:

* Docker service host
* Monitoring and metrics collection
* Log collection
* Local DNS-related services
* Self-hosted applications
* Local AI experimentation
* Remote access testing

### Synology DS1515+

The Synology NAS provides storage, backup planning, and additional Docker/VM experimentation. It is also used for media and photo-related workflows, external storage planning, and testing services that make more sense on the NAS side.

Known hardware/software details:

* Synology DS1515+
* DSM 7.1.1-42962 Update 9
* Intel Atom C2538, 4 cores
* 16 GB RAM
* RAID storage pool with approximately 21 TB usable space
* Connected to an APC UPS
* Tailscale enabled

Primary responsibilities:

* NAS storage
* Backup target and planning
* Docker services
* VM testing
* Photo and media workflows
* External drive workflows

---

## Core Services

| Service          | Purpose                          | Host                              | Current Notes                                                                                                            |
| ---------------- | -------------------------------- | --------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Pi-hole          | DNS filtering and DNS visibility | Dell Micro                        | Runs as a host-bound service/container setup. Used for local DNS filtering and troubleshooting client queries.           |
| Prometheus       | Metrics collection               | Dell Micro                        | Scrapes exporters and service metrics. Host port uses `9091` because `9090` is already occupied by a host-level service. |
| Grafana          | Dashboard visualization          | Dell Micro                        | Intended host mapping is `3001:3000` because port `3000` is already used by Open-WebUI.                                  |
| Loki             | Log storage backend              | Dell Micro                        | Receives logs from Promtail.                                                                                             |
| Promtail         | Log shipper                      | Dell Micro                        | Sends logs to Loki and includes a syslog listener for future network-device logging.                                     |
| cAdvisor         | Docker container metrics         | Dell Micro                        | Tracks container CPU, memory, and I/O metrics.                                                                           |
| Node Exporter    | Host metrics                     | Dell Micro                        | Provides system-level metrics such as CPU, RAM, disk, and network usage.                                                 |
| Pi-hole Exporter | Pi-hole metrics                  | Dell Micro                        | Exposes Pi-hole metrics for Prometheus.                                                                                  |
| Portainer Agent  | Docker management                | Dell Micro                        | Used for container visibility and management.                                                                            |
| Immich           | Self-hosted photo backup         | Dell Micro / NAS storage workflow | Used as a self-hosted family photo backup and iCloud-style alternative.                                                  |
| Open-WebUI       | Local AI web frontend            | Dell Micro                        | LAN-accessible frontend for local LLM testing.                                                                           |
| Ollama           | Local LLM backend                | Dell Micro                        | Provides local model API for Open-WebUI.  - Testing Phase Currently                                                                               |
| Home Assistant   | Home automation                  | Synology                          | Runs on the NAS side.                                                                                                    |
| iCloudPD         | iCloud photo sync                | Synology                          | Syncs iCloud photos into NAS storage. Current path: `/volume1/photo/icloud_james`.                                       |
| Portainer CE     | Docker management                | Synology                          | Used for NAS-side container management.                                                                                  |
| Wyze Bridge      | Camera bridge                    | Synology                          | Used for experimentation with camera/NVR workflows.                                                                      |

---

## Network and Access Notes

This environment is mostly LAN-first, with remote access handled carefully rather than exposing everything directly to the public internet.

Key design choices:

* Local services are primarily accessed over the home LAN.
* Tailscale is used for secure remote access testing and family access workflows.
* Public exposure is avoided unless there is a clear need and a safer access pattern.
* Port conflicts are tracked carefully because several services use common default ports.

Important port decisions:

* Open-WebUI uses host port `3000`.
* Grafana should use host port `3001`, mapped to container port `3000`.
* Prometheus uses host port `9091`, mapped to container port `9090`, because host port `9090` is already occupied.
* Loki uses `3100`.
* Immich uses `2283`.
* Ollama uses `11434`.
* Promtail syslog is planned on `1514`.

---

## Monitoring and Logs

Monitoring is one of the more important sections of this homelab because it connects multiple systems and turns the environment into something easier to observe and troubleshoot.

Current monitoring/logging stack:

* Prometheus for metrics collection
* Grafana for dashboards
* Loki for log storage
* Promtail for log shipping
* cAdvisor for Docker container metrics
* Node Exporter for host metrics
* Pi-hole Exporter for DNS-related metrics

Some issues encountered while building this stack:

* Prometheus configuration errors caused by YAML formatting mistakes.
* Host port conflicts, especially around `9090` and `3000`.
* Exporter configuration/version mismatch warnings during SNMP exporter testing.
* Docker container restart loops during bad config deployments.
* Inotify/watch exhaustion warnings during monitoring experiments.

## Storage and Backup Notes

Storage is split between the Dell Micro service host and the Synology NAS. The NAS is the main long-term storage system, while the Dell Micro handles active services.

Current backup approach:

* Synology provides the main local storage layer.
* AWS S3 with Glacier lifecycle rules provides low-cost long-term backup for photo/video media.
* Glacier transition is configured within 24 hours because this data is mostly archival and not frequently modified.
* Time Machine backups are provided through the Synology using a dedicated WD USB drive as the backup target.
* The Time Machine setup is intentionally simple so household devices can perform regular incremental backups without needing a complicated workflow.

## Remote Access

Remote access has been tested primarily with Tailscale. The goal is to allow secure access to services like Immich without making the whole environment public or relying on unsafe port forwarding.

Important lessons so far:

* Tailscale provides a private way to reach internal services without exposing them directly to the internet.
* It works well for Immich because devices can access the service both on the LAN and remotely.
* Mobile behavior matters, especially around VPN/DNS settings.
* Subnet routing and DNS settings can affect whether phones keep normal internet access.
* Family access needs to be simple enough that non-technical users can reliably use it.
* Immich remote access needs to balance convenience, privacy, and operational simplicity.

## Documentation

- [Monitoring Stack](docs/monitoring_stack_documentation.md) — Prometheus, Grafana, Loki, Promtail, cAdvisor, Node Exporter, Pi-hole Exporter, and SNMP Exporter.

