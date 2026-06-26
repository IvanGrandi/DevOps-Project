sudo apt-get update -y
sudo apt-get install -y fontconfig openjdk-21-jre
mkdir -p ~/.ssh
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
nano ~/.ssh/authorized_keys
mkdir -p /home/vagrant/jenkins-workspace
