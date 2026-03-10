Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/jammy64"

  # SCRIPT DE PROVISIONAMENTO
  provision_script = <<-SHELL
  sudo swapoff -a
  sudo sed -i '/ swap / s/^/#/' /etc/fstab

  echo "192.168.56.30 master" | sudo tee -a /etc/hosts
  echo "192.168.56.40 worker" | sudo tee -a /etc/hosts

  # gerar chave ssh se não existir
  if [ ! -f /home/vagrant/.ssh/id_rsa ]; then
      ssh-keygen -t rsa -N "" -f /home/vagrant/.ssh/id_rsa
  fi
  SHELL


  # MASTER
  config.vm.define "MASTER" do |master|
    master.vm.hostname = "master"
    master.vm.network "private_network", ip: "192.168.56.30"

    master.vm.provider "virtualbox" do |vb|
      vb.name = "MASTER"
      vb.memory = 2048
      vb.cpus = 2
    end

    master.vm.provision "shell", inline: provision_script
  end


  # WORKER
  config.vm.define "WORKER" do |worker|
    worker.vm.hostname = "worker"
    worker.vm.network "private_network", ip: "192.168.56.40"

    worker.vm.provider "virtualbox" do |vb|
      vb.name = "WORKER"
      vb.memory = 2048
      vb.cpus = 2
    end

    worker.vm.provision "shell", inline: provision_script
  end

end