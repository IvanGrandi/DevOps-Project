sudo apt-get update -y
sudo apt-get install -y docker.io
sudo systemctl enable --now docker
sudo usermod -aG docker vagrant
newgrp docker
docker ps
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
