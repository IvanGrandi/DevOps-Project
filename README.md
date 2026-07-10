# Project Documentation: Multi-VM CI/CD Pipeline Automation

Participants :

- Christopher Dubois
- Ivan Grandi
- Paul Leflon
- Antoine Cortial

## Project Overview

This project establishes an automated Continuous Integration and Continuous Deployment (CI/CD) infrastructure using two virtual machines managed by Vagrant.

This infrastructure aims to safely test and deploy a production instance of the **Jellyfin** media server using a completely automated pipeline.

### Architecture & Strategy

- **VM 1 (`jenkins-master`):** Acts as the CI/CD Controller. It orchestrates workflows, manages source code tracking, and holds configuration settings.
- **VM 2 (`podman-host`):** Acts as an isolated worker environment. It runs short-lived containers on-demand to test code, and hosts the final running web application using Podman's REST API.
- **VM 3 (`grafana-monitoring`):** Acts as the monitoring stack of our application. It deploys three infrastructure-tracking containers (Grafana, Prometheus, and Blackbox Exporter) via Docker Compose to supervise the environment and ensure the high availability and reliability of the deployed Jellyfin production application.
- **The Automation Flow:** When the pipeline runs, VM1 commands VM2 to automatically spin up a temporary worker container. Inside this isolated test environment, the project code is cloned, dependencies are pulled, and unit tests are executed. Once the tests pass, the verified software is deployed into a separate production container, its live web entry point is tested, and a summary alert is sent to a Discord channel. Immediately after the pipeline concludes, the temporary test container is completely destroyed to keep VM2 clean and free up system resources.

---

## Part 1: Infrastructure Provisioning

### VM 1: Jenkins Controller Setup

#### 1. Connect to the Jenkins Virtual Machine

```bash
vagrant ssh jenkins-master
```

Establishes a Secure Shell (SSH) connection to the Vagrant virtual machine named `jenkins-master`.

#### 2. Update Package Lists

```bash
sudo apt-get update -y
```

Resynchronizes package index files from upstream repositories to ensure access to the latest available software versions.

#### 3. Install Java Dependencies

```bash
sudo apt-get install -y fontconfig openjdk-21-jre
```

Installs `fontconfig` and OpenJDK 21 Java Runtime Environment (JRE), the foundational execution engine required to run the Jenkins server application.

#### 4. Download Jenkins GPG Key

```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
```

Downloads the official Jenkins GPG signing key to the system keyring, ensuring package authenticity and protection against code tampering before installation.

#### 5. Add Jenkins Repository to Sources List

```bash
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```

Appends the official stable Jenkins distribution platform channel directly into the host OS package system repository tracker.

#### 6. Install Jenkins

```bash
sudo apt-get update -y
sudo apt-get install -y jenkins
```

Forces an index reload to detect the new repository addition, then installs the core Jenkins controller services.

#### 7. Retrieve the Initial Administrator Password

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Displays the temporary unlock string required to initialize the server via its primary web configuration dashboard.

#### 8. Verification and Setup

Navigate to `localhost:8082` (or your configured forwarded host port) in a web browser to unlock the installation wizard, initialize core system components, and register your main user profile.

### VM 2: Podman Execution Host Setup

#### 1. Connect to the Podman Virtual Machine

```bash
vagrant ssh podman-host
```

Establishes an SSH connection to the second Vagrant virtual machine dedicated to container hosting workflows.

#### 2. Update Package Lists

```bash
sudo apt-get update -y
```

Resynchronizes repository tracking matrices across the worker operating system instance.

#### 3. Install Podman and Curl

```bash
sudo apt-get install -y podman curl
```

Deploys Podman (the daemonless container management tool used to instantiate independent job spaces) alongside `curl` to query endpoints and download scripts.

#### 4. Create the Podman API systemd Service File

```bash
sudo tee /etc/systemd/system/podman-api.service << 'EOF'
[Unit]
Description=Podman API Service Emulating Docker Engine
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/podman system service --time=0 tcp:0.0.0.0:2375
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```

Generates a system configuration utility that forces Podman to expose its internal operations as a REST API on network port `2375`. This enables the remote Jenkins Master to trigger engine tasks across the network.

#### 5. Reload the Systemd Manager Configuration

```bash
sudo systemctl daemon-reload
```

Forces the OS init subsystem manager to reload its active layout tree and pick up the new Podman engine tracking target.

#### 6. Enable and Start the Podman API Service

```bash
sudo systemctl enable --now podman-api.service
```

Ensures that the API pipeline turns on automatically during VM startup and initializes it immediately for current tasks.

#### 7. Pre-pull the Jenkins Inbound Agent Image

```bash
sudo podman pull docker.io/jenkins/inbound-agent:latest
```

Downloads the specialized inbound connection runner environment layer cache to the host storage system, saving execution time during future pipeline invocations.

#### 8. Verify the Network Socket and Service Status

```bash
sudo systemctl start podman-api.service
sudo systemctl status podman-api.service
sudo ss -tlnp | grep 2375
sudo podman ps
```

Runs a diagnostic series to confirm the API listener service is operational and properly attached to network port `2375`.

### VM 3: Grafana & Prometheus Monitoring Setup

#### 1. Connect to the Monitoring Virtual Machine

```bash
vagrant ssh grafana-monitoring
```

Establishes an SSH connection to the third Vagrant virtual machine dedicated to the deployed application monitoring.

#### 2. Update Package Lists & System Packages

```bash
sudo apt update && sudo apt upgrade -y
```

Resynchronizes package index files from upstream repositories and upgrades existing software to ensure system stability and security.

#### 3. Install Docker and Docker Compose

```bash
sudo apt install docker.io docker-compose -y
```

Deploys the Docker core container engine alongside the Docker Compose orchestration tool utility required to run multi-container applications.

#### 4. Create Configuration Directories

```bash
mkdir -p ~/monitoring/prometheus
cd ~/monitoring
```

Generates the required local structural directories on the host operating system to cleanly store configuration scripts.

#### 5. Initialize the Prometheus Configuration File

```bash
nano prometheus/prometheus.yml
```

Opens the text editor to define the custom scraping parameters and endpoints targets for the Prometheus monitoring engine.

```bash
global:
  scrape_interval: 15s

scrape_configs:

  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'jellyfin-status'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
        - http://192.168.56.21:8096
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115
```

Defines internal targets and instructs Prometheus to route HTTP network health probes toward the VM2 production server via the Blackbox Exporter proxy module.

#### 6. Create the Multi-Container Orchestration File

```bash
nano docker-compose.yml
```

Initializes the deployment workspace to construct the multi-container stack assembly configuration.

```bash
version: '3.8'

services:
  prometheus:
    container_name: prometheus
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    restart: unless-stopped

  blackbox-exporter:
    image: prom/blackbox-exporter:latest
    container_name: blackbox-exporter
    ports:
      - "9115:9115"
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana-storage:/var/lib/grafana
    restart: unless-stopped

volumes:
  grafana-storage:
```

Declares operational mappings, volume specifications, environment isolation parameters, and external port rules for the three monitoring services.

#### 7. Launch the Infrastructure Stack Containers

```bash
sudo docker-compose up -d
```

Parses the local docker-compose.yml blueprint file to download upstream image caches, mount configurations, and start all services concurrently in detached background mode.

#### 8. Verify Running Monitoring Services

```bash
sudo docker ps
```

## Executes a live status matrix scan to confirm that Grafana, Prometheus, and Blackbox Exporter containers are fully active, running, and healthy.

## Part 2: CI/CD Pipeline Workflow

The complete software delivery operation is managed via a Jenkins declarative pipeline divided into 5 distinct pipeline stages:

```
[Stage 1: Clone] ➔ [Stage 2: Provision SDK] ➔ [Stage 3: Run Tests] ➔ [Stage 4: Deploy Container] ➔ [Stage 5: Verify & Alert]
```

---

### Pipeline Stage Details

- **Stage 1: Clone Jellyfin Code:** Jenkins uses Git to pull down the code from the official repository into our container.
- **Stage 2: Provision Runtime SDK:** Downloader tools download and isolate a clean instance of the **.NET 10 SDK** needed to build and compile this specific app.
- **Stage 3: Run Functional Unit Tests:** Automated code validations test the component logic inside `src/Jellyfin.Extensions` to ensure no broken modifications move forward.
- **Stage 4: Production Deployment:** Jenkins interacts with the VM2 via the **Podman API over port 2375**. It forces the termination of old application versions, requests clean image pulls, sets up configurations, binds application ports (`8096`), and mounts long-term user storage paths.
- **Stage 5: Sanity Verification & Discord Notification:** The agent pauses some seconds to allow the server application routing configurations to initialize, then runs an active HTTP call verify request to `http://192.168.56.21:8096`. If it returns successfully, a styled update payload drops into a designated Discord channel. If any step fails, the system triggers an emergency alert to notify the development team.
