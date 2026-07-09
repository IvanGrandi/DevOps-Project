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

  config.vm.define "grafana-monitoring" do |grafana|
    grafana.vm.hostname = "grafana-monitoring"
    grafana.vm.box = "ubuntu/jammy64" 
    
    
    grafana.vm.network "private_network", ip: "192.168.56.22"
    
    grafana.vm.network "forwarded_port", guest: 3000, host: 3000, id: "grafana"
    grafana.vm.network "forwarded_port", guest: 9090, host: 9091, id: "prometheus"

    grafana.vm.provider "virtualbox" do |vb|
      vb.gui = false
      vb.memory = 2048
      vb.cpus = 1
      vb.name = "Grafana-Monitoring"
    end
  end
end