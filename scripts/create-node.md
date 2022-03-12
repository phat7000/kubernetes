# Cài đặt các node worker - kubernetes
Tạo thư mục kubernetes-centos7/worker1 và kubernetes-centos7/worker2 để cấu hình, tạo các file Vagrantfile trong thư mục tương ứng với nội dung

# kubernetes-centos7/worker1/Vagrantfile

#Vagrantfile

-*- mode: ruby -*-
vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.network "private_network", ip: "172.16.10.101"
  config.vm.hostname = "worker1.xtl"

  config.vm.provider "virtualbox" do |vb|
     vb.name = "worker1.xtl"
     vb.cpus = 1
     vb.memory = "1024"
  end
   
  config.vm.provision "shell", path: "./../install-docker-kube.sh"

  config.vm.provision "shell", inline: <<-SHELL
  
    echo "root password"
    echo "123" | passwd --stdin root
    sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
    systemctl reload sshd


cat >>/etc/hosts<<EOF
172.16.10.100 master.xtl
172.16.10.101 worker1.xtl
172.16.10.102 worker2.xtl
EOF

    
  SHELL
end

# kubernetes-centos7/worker2/Vagrantfile

-*- mode: ruby -*-
vi: set ft=ruby :

-*- mode: ruby -*-
vi: set ft=ruby :
  
Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.network "private_network", ip: "172.16.10.102"
  config.vm.hostname = "worker2.xtl"

  config.vm.provider "virtualbox" do |vb|
     vb.name = "worker2.xtl"
     vb.cpus = 1
     vb.memory = "1024"
  end
   
  config.vm.provision "shell", path: "./../install-docker-kube.sh"

  config.vm.provision "shell", inline: <<-SHELL
  
    echo "root password"
    echo "123" | passwd --stdin root
    sed -i 's/^PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
    systemctl reload sshd
    

cat >>/etc/hosts<<EOF
172.16.10.100 master.xtl
172.16.10.101 worker1.xtl
172.16.10.102 worker2.xtl
EOF


  SHELL
end


Sau đó vào từng thư mục, thực hiện lệnh vagrant up để tạo hai máy ảo có cài đặt docker và kubernetes,
máy ảo có tên và ip tương ứng worker1.xtl (172.16.10.101), worker2.xtl (172.16.10.102)

# Kết nối Node vào Cluster

Hãy vào máy node master (bằng SSH ssh root@172.16.10.100). Thực hiện lệnh sau với Cluster để lấy lệnh kết nối

kubeadm token create --print-join-command

Nó cho nội dung lệnh kubeadm join ... thực hiện lệnh này trên các node worker thì node worker sẽ nối vào Cluster

SSH vào máy worker1, work2 và thực hiện kết nối

# Giờ kiểm tra các node có trong Cluster
kubectl get nodes
