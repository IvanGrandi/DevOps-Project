# 1. Update your apt mirror indexes

sudo apt-get update -y

# 2. Install Podman (No Java or Fontconfig required inside the host anymore!)

sudo apt-get install -y podman curl

# 3. Create the automated systemd unit file for the Podman REST API

# This exposes the engine interface on port 2375 to answer Jenkins' orchestration calls.

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

# 4. Refresh your background systemd configuration manager

sudo systemctl daemon-reload

# 5. Enable the Podman engine service to start automatically on system boot

sudo systemctl enable --now podman-api.service

# 6. Pre-pull the inbound agent image into Podman's local asset matrix cache

# This prevents Jenkins from timing out during your first pipeline run.

sudo podman pull docker.io/jenkins/inbound-agent:latest

# Verify status

sudo systemctl start podman-api.service
sudo systemctl status podman-api.service
sudo ss -tlnp | grep 2375
