#Cách 1:
Run trực tiếp từ docker windows, macos,

#Cách 2:
Tạo Cluster Kubernetes hoàn chỉnh

Tạo ra một Cluster Kubernetes hoàn chỉnh từ 3 máy (3 VPS - hay 3 Server) chạy CentOS, ... bạn có thể dùng cách này khi triển khai môi trường product.

Hệ thống này gồm:
Tên máy/Hostname 	Thông tin hệ thống 												Vai trò
master.xtl 			HĐH CentOS7, Docker CE, Kubernetes. Địa chỉ IP 172.16.10.100 	Khởi tạo là master
worker1.xtl 		HĐH CentOS7, Docker CE, Kubernetes. Địa chỉ IP 172.16.10.101 	Khởi tạo là worker
worker2.xtl 		HĐH CentOS7, Docker CE, Kubernetes. Địa chỉ IP 172.16.10.102 	Khởi tạo là worker

Bạn có thể tải về hệ điều hành CentOS