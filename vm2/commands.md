sudo apt-get update -y
sudo apt-get install -y fontconfig openjdk-21-jre
mkdir -p ~/.ssh
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
nano ~/.ssh/authorized_keys
mkdir -p /home/vagrant/jenkins-workspace

sudo apt-get install -y docker.io
sudo systemctl enable --now docker
sudo usermod -aG docker vagrant
newgrp docker

sudo mkdir -p /etc/systemd/system/docker.service.d
sudo nano /etc/systemd/system/docker.service.d/override.conf
sudo systemctl daemon-reload
sudo systemctl restart docker
vagrant@ubuntu-jammy:~$ sudo nano /etc/systemd/system/docker.service.d/override.conf
vagrant@ubuntu-jammy:~$ sudo nano /etc/systemd/system/docker.service.d/override.conf
vagrant@ubuntu-jammy:~$ sudo nano /etc/systemd/system/docker.service.d/override.conf
vagrant@ubuntu-jammy:~$ sudo mkdir -p /opt/jenkins-agent/.ssh
vagrant@ubuntu-jammy:~$ sudo nano /opt/jenkins-agent/.ssh/authorized_keys
vagrant@ubuntu-jammy:~$ sudo chmod 700 /opt/jenkins-agent/.ssh
vagrant@ubuntu-jammy:~$ sudo chmod 600 /opt/jenkins-agent/.ssh/authorized_keys
vagrant@ubuntu-jammy:~$ sudo chown -R 1000:1000 /opt/jenkins-agent/
vagrant@ubuntu-jammy:~$ docker pull jenkins/ssh-agent:latest-jdk21
