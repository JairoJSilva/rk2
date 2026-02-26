Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64" # Ubuntu 22.04 LTS

  # Configuração do Master (Control Plane)
  config.vm.define "rke2-master" do |master|
    master.vm.hostname = "rke2-master"
    master.vm.network "private_network", ip: "192.168.56.10"
    master.vm.provider "virtualbox" do |v|
      v.memory = 3072
      v.cpus = 2
      v.name = "rke2-master"
    end
  end

  # Configuração do Worker (Agente)
  config.vm.define "rke2-worker" do |worker|
    worker.vm.hostname = "rke2-worker"
    worker.vm.network "private_network", ip: "192.168.56.11"
    worker.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 1
      v.name = "rke2-worker"
    end
  end

  # Ajuste de DNS para evitar problemas de resolução interna
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update && apt-get install -y curl
    echo "192.168.56.10 rke2-master" >> /etc/hosts
    echo "192.168.56.11 rke2-worker" >> /etc/hosts
  SHELL
end
