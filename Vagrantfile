Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"

  # ==========================================
  # VM1: Pipeline Orchestrator (Jenkins Master)
  # ==========================================
  config.vm.define "jenkins-master" do |master|
    master.vm.hostname = "jenkins-master"
    
    # Network configurations from your original VM1 file
    master.vm.network "private_network", ip: "192.168.56.20"
    master.vm.network "forwarded_port", guest: 8080, host: 8082, id: "jenkins"
    master.vm.network "forwarded_port", guest: 22, host: 2225, id: "ssh"

    master.vm.provider "virtualbox" do |vb|
      vb.gui = false 
      vb.memory = 2048
      vb.cpus = 2
      vb.name = "Jenkins-Master"
    end
  end

  # ==========================================
  # VM2: Pipeline Agent & Production Server (Podman)
  # ==========================================
  config.vm.define "podman-host" do |podman|
    podman.vm.hostname = "podman-host"
    
    podman.vm.network "private_network", ip: "192.168.56.21"
    podman.vm.network "forwarded_port", guest: 8080, host: 8083
    podman.vm.network "forwarded_port", guest: 22, host: 2226, id: "ssh"
    podman.vm.network "forwarded_port", guest: 8096, host: 8096, id: "jellyfin"

    podman.vm.provider "virtualbox" do |vb|
      vb.gui = false
      vb.memory = 2048 
      vb.cpus = 2
      vb.name = "Podman-Host"
    end
  end
end