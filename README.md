# k8s

系统:ubuntu 20.42 server

规划: master-192.168.1.230
     node1-192.168.1.231
     node2-192.168.1.232

系统初始化(包括master和node):
     1.关闭防火墙
     2.关闭swap vim /etc/fstab  注释swap相关永久关闭
     3.设置主机名字 hostnamectl set-hostname 
     4.master和node添加/etc/hosts ip和主机名对应关系
     5.将桥接的 IPv4 流量传递到 iptables 的链 
	cat > /etc/sysctl.d/k8s.conf << EOF net.bridge.bridge-nf-call-ip6tables =1 			
	net.bridge.bridge-nf-call-iptables = 1 
	EOF 
       sudo sysctl --system 
     6.时间同步
     7.安装docker kubelet kubeadm(master和node)
		sudo apt update sudo apt install docker.io sudo systemctl start docker sudo         systemctl enable docker
               
sudo apt-get update && sudo apt-get install -y ca-certificates curl software-properties-common apt-transport-https curl 
curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add - sudo tee /etc/apt/sources.list.d/kubernetes.list <<EOF deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main 
EOF 
sudo apt-get update 
sudo apt-get install -y kubelet kubeadm kubectl 
sudo apt-mark hold kubelet kubeadm kubectl 

     8.初始化master :
             kubeadm init \
--apiserver-advertise-address=192.168.1.230 \
--image-repository registry.aliyuncs.com/google_containers \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16
镜像下载失败的,可以手动dockerhub下载同版本的该tag
 
     9.mkdir -p $HOME/.kube sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config sudo chown $(id -u):$(id -g) $HOME/.kube/config 
 
     10.安装cni插件(使用calico)
             kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
     11:查看工作节点是否准备完成全部running状态,全部running后集群搭建成功
             kubectl get pods -n kube-system 
踩坑记录:
              dockers引擎修改为systemd
