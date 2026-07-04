## Commands to launch Vm1 with Jenkins

### 1. Connect to the Jenkins Virtual Machine

```bash
vagrant ssh jenkins-master
```

### 2. Update Package Lists

```bash
sudo apt-get update -y
```

### 3. Install Java Dependencies

```bash
sudo apt-get install -y fontconfig openjdk-21-jre
```

### 4. Download Jenkins GPG Key

```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
```

Downloads the official Jenkins GPG signing key and saves it to the system's keyring. This key verifies the integrity and authenticity of the Jenkins packages before installation.

### 5. Add Jenkins Repository to Sources List

```bash
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
```

Adds the official Jenkins stable repository to the system's package source list, explicitly referencing the downloaded GPG key for secure package verification.

### 6. Install Jenkins

```bash
sudo apt-get update -y
sudo apt-get install -y jenkins
```

Updates the package lists again to include the newly added Jenkins repository, and then proceeds to download and install the Jenkins service.

### 7. Retrieve the Initial Administrator Password

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

```

### 8. Verification and Setup

Go to localhost:8082 to test jenkins, create an account, and use the admin password
