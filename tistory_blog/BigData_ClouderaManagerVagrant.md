# Vagrant 란?
<div style="margint-bottom:5px;padding:7px; background-color:#1563FF;color:white">
Vagrant is a tool for building and managing virtual machine environments in a single workflow. With an easy-to-use workflow and focus on automation, Vagrant lowers development environment setup time, increases production parity, and makes the "works on my machine" excuse a relic of the past.
</div>

- 단일 workflow에서 가상머신 환경들을 만들고 관리하는 Tool
- 사용하기 쉬운 workflow와 자동화에 초점을 맞췄고, 개발 환경 셋팅 시간을 줄여줌

# Vagrant 설치
### 아래 링크로 가서 다운로드 후 설치
👉 [Download - Vagrant by HashiCorp](https://www.vagrantup.com/downloads.html) 

### vagrant workspace 폴더 생성
```bash
$> mkdir ~/vagrant_workspace
$> cd ~/vagrant_workspace
```

# Vagrant를 활용하여 Cloudera Manager 설치하기

### `Vagrantfile` 만들기
```bash
$> vi ~/vagrant_workspace/Vagrantfile

### Cloudera Agent 스크립트
$agent_script = <<SCRIPT

#Disable selinux:
sed -i 's/^\(SELINUX\s*=\s*\).*$/\1disabled/' /etc/selinux/config

#Setup NTP:
yum -y install ntp
chkconfig ntpd on
service ntpd start
sudo hwclock --systohc

#Disable Firawall:
systemctl stop firewalld
systemctl disable firewalld

SCRIPT

### Cloudera Manager 스크립트
$manager_script = <<SCRIPT

wget -O /etc/yum.repos.d/cloudera-manager.repo https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo

yum -y update

yum -y install oracle-j2sdk1.7 cloudera-manager-server cloudera-manager-daemons cloudera-manager-server-db-2

service cloudera-scm-server-db start
service cloudera-scm-server start

SCRIPT

$hosts_script = <<SCRIPT

cat > /etc/hosts <<EOF
127.0.0.1       localhost
EOF

SCRIPT

Vagrant.configure("2") do |config|

	# Define base image
	config.vm.box = "bento/centos-7.2"

	# Manage /etc/hosts on host and VMs
	config.hostmanager.enabled = false
	config.hostmanager.manage_host = true
	config.hostmanager.include_offline = true
	config.hostmanager.ignore_private_ip = false

	config.vm.define :master do |master|
		master.vm.provider :virtualbox do |v|
			v.name = "taco"
			v.customize ["modifyvm", :id, "--memory", "8192"]
		end
		master.vm.network :private_network, ip: "192.168.57.111"
		master.vm.hostname = "taco"
		master.vm.provision :shell, :inline => $agent_script
		master.vm.provision :shell, :inline => $hosts_script
		master.vm.provision :hostmanager
		master.vm.provision :shell, :inline => $manager_script
	end

	config.vm.define :node2 do |node2|
		node2.vm.provider :virtualbox do |v|
			v.name = "burger"
			v.customize ["modifyvm", :id, "--memory", "1248"]
		end
		node2.vm.network :private_network, ip: "192.168.57.112"
		node2.vm.hostname = "burger"
		node2.vm.provision :shell, :inline => $agent_script
		node2.vm.provision :shell, :inline => $hosts_script
		node2.vm.provision :hostmanager
	end
	config.vm.define :node3 do |node3|
		node3.vm.provider :virtualbox do |v|
			v.name = "pizza"
			v.customize ["modifyvm", :id, "--memory", "1248"]
		end
		node3.vm.network :private_network, ip: "192.168.57.113"
		node3.vm.hostname = "pizza"
		node3.vm.provision :shell, :inline => $agent_script
		node3.vm.provision :shell, :inline => $hosts_script
		node3.vm.provision :hostmanager
	end
	config.vm.define :node4 do |node4|
		node4.vm.provider :virtualbox do |v|
			v.name = "cookies"
			v.customize ["modifyvm", :id, "--memory", "1248"]
		end
		node4.vm.network :private_network, ip: "192.168.57.114"
		node4.vm.hostname = "cookies"
		node4.vm.provision :shell, :inline => $agent_script
		node4.vm.provision :shell, :inline => $hosts_script
		node4.vm.provision :hostmanager
	end
	config.vm.define :node5 do |node5|
		node5.vm.provider :virtualbox do |v|
			v.name = "smoothies"
			v.customize ["modifyvm", :id, "--memory", "1248"]
		end
		node5.vm.network :private_network, ip: "192.168.57.115"
		node5.vm.hostname = "smoothies"
		node5.vm.provision :shell, :inline => $agent_script
		node5.vm.provision :shell, :inline => $hosts_script
		node5.vm.provision :hostmanager
	end
end

```

### Vagrant 시작
```bash
$> vagrant up
```

#### 🙉 유의사항 : `root` 계정의 비밀번호는 `vagrant`

## 참고자료
[Introduction - Vagrant by HashiCorp](https://www.vagrantup.com/intro/index.html)  
[INSTALLING HADOOP CLUSTER WITH CLOUDERA MANAGER](https://www.softserveinc.com/en-us/blogs/hadoop-cluster-cloudera-manager/)  
[[Vagrant] 설치 및 기초 사용방법 - Windows](https://ossian.tistory.com/86)