sudo apt-get update -y

sudo apt-get install -y fontconfig openjdk-21-jre

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update -y
sudo apt-get install -y jenkins

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

### Go to local host 8082 to test jenkins, create an account, and use the admin password

ssh-keygen -t rsa -b 4096 -N "" -f ~/.ssh/id_rsa

cat ~/.ssh/id_rsa.pub

### We copy the key inside the Vm2 and Vm3 with nano

### Test the conection

vagrant@ubuntu-jammy:~$ ssh -o StrictHostKeyChecking=no vagrant@192.168.56.21 "echo VM2 Connection Successful"
VM2 Connection Successful
vagrant@ubuntu-jammy:~$ ssh -o StrictHostKeyChecking=no vagrant@192.168.56.22 "echo VM3 Connection Successful"
Warning: Permanently added '192.168.56.22' (ED25519) to the list of known hosts.
VM3 Connection Successful
