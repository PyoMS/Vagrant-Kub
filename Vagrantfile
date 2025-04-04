Vagrant.configure("2") do |config|
  config.vm.box = "generic/rocky9"

  # 브릿지 어댑터 + 고정 IP
  config.vm.network "public_network", ip: "192.168.0.100"

  config.vm.provider "virtualbox" do |vb|
    vb.name = "rocky9-k8s-dev"
    vb.memory = 4096
    vb.cpus = 2
  end

  config.vm.provision "shell", inline: <<-SHELL
    # 시스템 업데이트
    dnf update -y

    # Docker 설치
    dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
    dnf install -y docker-ce docker-ce-cli containerd.io
    systemctl enable --now docker

    # br_netfilter 및 sysctl 설정 (K8s 준비)
    cat <<EOF | tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

    modprobe br_netfilter

    cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

    sysctl --system

    # Kubernetes 저장소 등록
    cat <<EOF | tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF

    # Kubernetes 설치
    dnf install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
    systemctl enable --now kubelet
  SHELL
end
