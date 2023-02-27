# Cập nhật 3.2022

# Tạo máy Master Kubernetes

kubernetes-centos7/install-docker-kube.sh

!/bin/bash

# Cai dat Docker
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum update -y && yum install docker-ce-18.06.2.ce -y
usermod -aG docker $(whoami)

## Create /etc/docker directory.
mkdir /etc/docker

## Setup daemon.
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF

mkdir -p /etc/systemd/system/docker.service.d


## Restart Docker
systemctl enable docker.service
systemctl daemon-reload
systemctl restart docker


## Tat SELinux
setenforce 0
sed -i --follow-symlinks 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux

## Tat Firewall
systemctl disable firewalld >/dev/null 2>&1
systemctl stop firewalld

## sysctl
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system >/dev/null 2>&1

## Tat swap
sed -i '/swap/d' /etc/fstab
swapoff -a

## Add yum repo file for Kubernetes
cat >>/etc/yum.repos.d/kubernetes.repo<<EOF
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

yum install -y -q kubeadm kubelet kubectl

systemctl enable kubelet
systemctl start kubelet

## Configure NetworkManager before attempting to use Calico networking.
cat >>/etc/NetworkManager/conf.d/calico.conf<<EOF
[keyfile]
unmanaged-devices=interface-name:cali*;interface-name:tunl*
EOF
EOF

# Thiết lập file chạy được
chmode +x install-docker-kube.sh

# Khởi tạo Cluster
Trong lệnh khởi tạo cluster có tham số --pod-network-cidr để chọn cấu hình mạng của POD, do dự định dùng Addon calico nên chọn --pod-network-cidr=192.168.0.0/16

# Khởi tạo là nút master của Cluster
kubeadm init --apiserver-advertise-address=172.16.10.100 --pod-network-cidr=192.168.0.0/16

# Sau khi lệnh chạy xong
chạy tiếp cụm lệnh nó yêu cầu chạy sau khi khởi tạo- để chép file cấu hình đảm bảo trình kubectl trên máy này kết nối Cluster

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config


# Tiếp đó,
nó yêu cầu cài đặt một Plugin mạng trong các Plugin tại addon, ở đây đã chọn calico, nên chạy lệnh sau để cài nó
kubectl apply -f https://docs.projectcalico.org/v3.10/manifests/calico.yaml


# Gõ vài lệnh sau để kiểm tra

## Thông tin cluster
kubectl cluster-info
## Các node trong cluster
kubectl get nodes
## Các pod đang chạy trong tất cả các namespace
kubectl get pods -A

Vậy là đã có Cluster với 1 node!

# Cấu hình kubectl máy trạm truy cập đến các Cluster
Chương trình client kubectl là công cụ dòng lệnh kết nối và tương tác với các Cluster Kubernetes,
thường khi cài đặt Kubernetes mọi người cũng cài luôn kubectl như phần trên trên, ngay cả máy cài Docker Desktop cũng đã có kubectl.
Tất nhiên, bạn có cài đặt kubectl trên một máy không Docker, không Kubernetes với mục đích chỉ dùng nó kết nối đến hệ thống Cluster từ xa. 
Nếu muốn cài ở máy độc lập như vậy xem tại Intall kubectl

## File cấu hình lệnh kubectl
Khi thi hành kubectl, thì nó đọc file cấu hình ở đường dẫn $HOME/.kube/config để biết các thông số để kết nối đến Cluster.
($HOME là thư mục gốc dành cho user đang chạy, để biết chính xác gõ lệnh echo $HOME) - tài khoản root thì đó là /root/.kube/config

## Trở lại máy Host, để xem nội dung cấu hình kubectl gõ lệnh
kubectl config view

Tại máy master ở trên, có file cấu hình cho tại /root/.kube/config, ta copy file cấu hình này ra lưu thành file config-mycluster (không ghi đè vào config hiện tại của máy HOST)
scp root@172.16.10.100:/etc/kubernetes/admin.conf ~/.kube/config-mycluster
(Nhớ thay đường dẫn theo user của bạn)

Vậy trên máy của tôi đang có 2 file cấu hình

    /User/abc/.kube/config-mycluster cấu hình kết nối đến Cluster mới tạo ở trên
    /User/abc/.kube/config cấu hình kết nối đến Cluster cục bộ của bản Kubernetes có sẵn của Docker

Nếu muốn yêu cầu kubectl sử dụng ngay file cấu hình nào đó, thì gán biến môi trường KUBECONFIG bằng đường dẫn file cấu hình, ví dụ sử dụng file cấu hình config-mycluster 
export KUBECONFIG=/Users/xuanthulab/.kube/config-mycluster

Sau lệnh đó thì kubectl sẽ dùng config-mycluster để có thông tin kết nối đến, nhưng trường hợp này chỉ có hiệu lực trong một phiên làm việc,
ví dụ nếu bạn đóng terminal và mở lại thì lại phải thiết lập lại biến môi trường như trên.

## Sử dụng các context trong cấu hình kubectl
(hãy tắt terminal và mở lại để KUBECONFIG không còn tác dụng)
Khi bạn xem nội dung config với lệnh kubectl config view, bạn thấy rằng nó khai báo có các mục cluster là thông tin của cluster với tên,
user thông tin user được đăng nhập, context là ngữ cảnh sử dụng, mỗi ngữ cảnh có tên trong đó có thông tin user và cluster.

Ở file trên bạn thấy mục current-context là context với tên docker-desktop, có nghĩa là kết nối đến cluster có tên docker-desktop với user là docker-desktop
Giờ bạn sẽ thực hiện kết hợp 2 file: config và config-mycluster thành 1 và lưu trở lại config.

export KUBECONFIG=~/.kube/config:~/.kube/config-mycluster
kubectl config view --flatten > ~/.kube/config_temp
mv ~/.kube/config_temp ~/.kube/config

Như vậy trong file cấu hình đã có các ngữ cảnh khác nhau để sử dụng, đóng terminal và mở lại rồi gõ lệnh, có các ngữ cảnh nào

Ký hiệu * là cho biết context hiện tại, nếu muốn chuyển làm việc sang context có tên kubernetes-admin@kubernetes (nối với cluster mới tạo ở trên) thì gõ lệnh

kubectl config use-context kubernetes-admin@kubernetes

Như vậy sử dụng context, giúp bạn lưu và chuyển đổi dễ dàng các loại kết nối đến các cluster của bạn