Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"

  WORKER_COUNT = 2

  # =========================
  # MASTER
  # =========================
  config.vm.define "rke2-master" do |master|
    master.vm.hostname = "rke2-master"
    master.vm.network "private_network", ip: "192.168.56.10"

    master.vm.provider "virtualbox" do |v|
      v.memory = 6144
      v.cpus = 3
    end

    master.vm.provision "shell", inline: <<-SHELL
      set -e

      # Base Kubernetes requirements
      swapoff -a
      sed -i '/ swap / s/^/#/' /etc/fstab

      modprobe overlay
      modprobe br_netfilter

      cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables=1
net.ipv4.ip_forward=1
EOF
      sysctl --system

      apt-get update
      apt-get install -y curl

      # Install RKE2 Server
      curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE="server" sh -

      systemctl enable rke2-server
      systemctl start rke2-server

      echo "Aguardando cluster subir..."
      sleep 45

      export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
      echo "export KUBECONFIG=/etc/rancher/rke2/rke2.yaml" >> /root/.bashrc

      # Salva token para workers
      cp /var/lib/rancher/rke2/server/node-token /vagrant/node-token

      # Instala Helm
      curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

      # Adiciona repositório Rancher
      helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
      helm repo update

      # Namespace
      kubectl create namespace cattle-system

      # Instala Cert-Manager
      kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
      echo "Aguardando cert-manager..."
      sleep 60

      # Instala Rancher
      helm install rancher rancher-latest/rancher \
        --namespace cattle-system \
        --set hostname=rancher.local \
        --set bootstrapPassword=admin

      echo "Instalação finalizada!"
    SHELL
  end

  # =========================
  # WORKERS AUTOMÁTICOS
  # =========================
  (1..WORKER_COUNT).each do |i|
    config.vm.define "rke2-worker-#{i}" do |worker|
      worker.vm.hostname = "rke2-worker-#{i}"
      worker.vm.network "private_network", ip: "192.168.56.#{10 + i}"

      worker.vm.provider "virtualbox" do |v|
        v.memory = 3072
        v.cpus = 2
      end

      worker.vm.provision "shell", inline: <<-SHELL
        set -e

        swapoff -a
        sed -i '/ swap / s/^/#/' /etc/fstab

        modprobe overlay
        modprobe br_netfilter

        apt-get update
        apt-get install -y curl

        while [ ! -f /vagrant/node-token ]; do
          sleep 5
        done

        TOKEN=$(cat /vagrant/node-token)

        curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE="agent" sh -

        mkdir -p /etc/rancher/rke2

        cat <<EOF | tee /etc/rancher/rke2/config.yaml
server: https://192.168.56.10:9345
token: ${TOKEN}
EOF

        systemctl enable rke2-agent
        systemctl start rke2-agent
      SHELL
    end
  end
end