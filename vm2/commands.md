## Commands to launch Vm2 with Podman

### 1. Connect to the Podman Virtual Machine

```bash
vagrant ssh podman-host
```

### 2. Update Package Lists

```bash
sudo apt-get update -y
```

### 3. Install Podman and Curl

```bash
sudo apt-get install -y podman curl
```

Installs Podman (the daemonless container engine used to run Jenkins agents) and `curl` for making web requests and health checks.

### 4. Create the Podman API systemd Service File (Corrected Syntax)

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

Creates a background system service configuration file. This exposes Podman’s REST API over network port `2375`, allowing the Jenkins master on the other VM to communicate with it using standard Docker API commands.

---

### 5. Reload the Systemd Manager Configuration

```bash
sudo systemctl daemon-reload
```

Forces systemd to scan for new or changed service configuration units, ensuring it registers the newly created `podman-api.service`.

### 6. Enable and Start the Podman API Service

```bash
sudo systemctl enable --now podman-api.service
```

Configures the Podman API service to launch automatically whenever the virtual machine boots up, and immediately starts the service right now.

### 7. Pre-pull the Jenkins Inbound Agent Image

```bash
sudo podman pull docker.io/jenkins/inbound-agent:latest
```

Downloads the Jenkins agent image from Docker Hub into Podman's local cache ahead of time. This prevents Jenkins pipelines from timing out while waiting for a large download during their first run.

### 8. Verify the Network Socket and Service Status

```bash
sudo systemctl start podman-api.service
sudo systemctl status podman-api.service
sudo ss -tlnp | grep 2375
sudo podman ps
```

Runs verification steps to check that the API service is active, running without errors, successfully listening on TCP port `2375`, and capable of responding to container tracking commands (`podman ps`).
