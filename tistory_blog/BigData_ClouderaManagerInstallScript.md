> IP만 달려있는 Linux 서버(CentOS 6.7)들이 있을 때, Cloudera 설치를 쉽게 할 수 있는 스크립트를 만들어 보았습니다 ^^

# IP만 있는 CentOS 서버 만들기
- VirtualBox와 Vagrant 조합으로 4대의 가상머신을 생성  
  (메모리 여유가 된다면 가상머신을 더 만들어도 됨)
- `node1` 에 cloudera-manager-server와 cloudera manager용 db(mysql)가 설치될 예정 
    ```bash
    Vagrant.configure("2") do |config|

        # Define base image
        config.vm.box = "bento/centos-6.7"

        # Manage /etc/hosts on host and VMs
        config.hostmanager.enabled = false
        config.hostmanager.manage_host = true
        config.hostmanager.include_offline = true
        config.hostmanager.ignore_private_ip = false

        config.vm.define :node1 do |node1|
            node1.vm.provider :virtualbox do |v|
                v.name = "node1"
                v.customize ["modifyvm", :id, "--memory", "8192"]
            end
            node1.vm.network :private_network, ip: "192.168.57.111"
        end

        config.vm.define :node2 do |node2|
            node2.vm.provider :virtualbox do |v|
                v.name = "node2"
                v.customize ["modifyvm", :id, "--memory", "1248"]
            end
            node2.vm.network :private_network, ip: "192.168.57.112"
        end
        config.vm.define :node3 do |node3|
            node3.vm.provider :virtualbox do |v|
                v.name = "node3"
                v.customize ["modifyvm", :id, "--memory", "1248"]
            end
            node3.vm.network :private_network, ip: "192.168.57.113"
        end
        config.vm.define :node4 do |node4|
            node4.vm.provider :virtualbox do |v|
                v.name = "node4"
                v.customize ["modifyvm", :id, "--memory", "1248"]
            end
            node4.vm.network :private_network, ip: "192.168.57.114"
        end
    end
    ```

# Cloudera Manager 설치 스크립트
- `node1`에 설치 스크립트 (`cm_install.sh`)와 mysql 설정 파일 (`my.cnf`)을 준비  
  (ftp 툴을 사용하여 전송하거나 ssh 접속 후 vi 명령어를 통해 직접 작성하면 됨)

- 설치 스크립트 (`cm_install.sh`)
    ```bash
    #!/bin/bash
    #*** my.cnf 파일 준비 필요 ***

    genSshKeyScript() {
      rm -rf $HOME/.ssh

      echo -ne '\n' | echo -ne '\n' | echo -ne '\n' | ssh-keygen -t rsa
      ls -al ~/.ssh/

      # 권한 설정
      sudo chmod 700 ~/.ssh
      sudo chmod 600 ~/.ssh/id_rsa
      sudo chmod 644 ~/.ssh/id_rsa.pub
    }

    agentScript() {
      sysctl vm.swappiness=10
      echo "vm.swappiness=10">> /etc/sysctl.conf

      echo never > /sys/kernel/mm/transparent_hugepage/defrag
      echo never > /sys/kernel/mm/transparent_hugepage/enabled
      echo "echo never > /sys/kernel/mm/transparent_hugepage/defrag" >> /etc/rc.local
      echo "echo never > /sys/kernel/mm/transparent_hugepage/enabled" >> /etc/rc.local
      
      #selinux 비활성화 
      sed -i 's/^\(SELINUX\s*=\s*\).*$/\1disabled/' /etc/selinux/config

      #NTP 설정
      yum -y install ntp 
      chkconfig ntpd on 
      service ntpd start 
      hwclock --systohc 
      
      mv /etc/localtime /etc/localtime_org
      ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime
      date

      #방화벽 해제
      chkconfig iptables off

      # training 계정 생성
      useradd training
      expect -c "set timeout 5
      spawn passwd training
      expect \"New password:\"
      send \"training\r\"
      expect \"Retype new password:\"
      send \"training\r\"
      expect eof"

      chmod 640 /etc/sudoers
      echo 'training    ALL=(ALL)    NOPASSWD:ALL' >> /etc/sudoers
      chmod 440 /etc/sudoers
    }

    managerScript() {
      wget -O /etc/yum.repos.d/cloudera-manager.repo https://archive.cloudera.com/cm5/redhat/6/x86_64/cm/cloudera-manager.repo 
      
      yum -y update
      yum -y install oracle-j2sdk1.7 cloudera-manager-server cloudera-manager-daemons 
      
      # MariaDB repo 설정
      #echo "[mariadb]
      #name = MariaDB
      #baseurl = http://yum.mariadb.org/5.5/rhel6-amd64
      #gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
      #gpgcheck=1" > /etc/yum.repos.d/mariadb.repo

      # MY SQL Install
      yum -y install mysql-server mysql
      chkconfig mysqld on
      service mysqld start
      
      #run mysql_secure_installation
      yum -y install expect
      
      SECURE_MYSQL=$(expect -c "
      set timeout 10
      spawn mysql_secure_installation
      expect \"Enter current password for root (enter for none):\"
      send \"\r\"
      expect \"Change the root password?\"
      send \"n\r\"
      expect \"Remove anonymous users?\"
      send \"y\r\"
      expect \"Disallow root login remotely?\"
      send \"n\r\"
      expect \"Remove test database and access to it?\"
      send \"y\r\"
      expect \"Reload privilege tables now?\"
      send \"y\r\"
      expect eof
      ")
      
      echo "$SECURE_MYSQL"
      
      mysql -u root -e "create database scm" mysql
      mysql -u root -e "grant all on *.* to 'scm'@'%' identified by 'scm' with grant option;" mysql
      
      mysql -u root -e "create database rman" mysql
      mysql -u root -e "grant all on rman.* to 'rman'@'%' identified by 'rman' with grant option;" mysql

      mysql -u root -e "create database hue" mysql
      mysql -u root -e "grant all on hue.* to 'hue'@'%' identified by 'hue' with grant option;" mysql

      mysql -u root -e "create database metastore" mysql
      mysql -u root -e "grant all on metastore.* to 'hive'@'%' identified by 'hive' with grant option;" mysql

      mysql -u root -e "create database oozie" mysql
      mysql -u root -e "grant all on oozie.* to 'oozie'@'%' identified by 'oozie' with grant option;" mysql
      
      # JDBC Driver Install
      wget https://cdn.mysql.com//Downloads/Connector-J/mysql-connector-java-5.1.47.tar.gz
      tar zxvf mysql-connector-java-5.1.47.tar.gz
      mkdir -p /usr/share/java/
      cp mysql-connector-java-5.1.47/mysql-connector-java-5.1.47-bin.jar /usr/share/java/.
      ln -s /usr/share/java/mysql-connector-java-5.1.47-bin.jar /usr/share/java/mysql-connector-java.jar
      
      # Preparing the Cloudera Manager Server Database
      /usr/share/cmf/schema/scm_prepare_database.sh mysql -h localhost scm scm scm
      
      service cloudera-scm-server start

      tail -f /var/log/cloudera-scm-server/cloudera-scm-server.log
    }

    wget https://dl.fedoraproject.org/pub/epel/6/x86_64/Packages/s/sshpass-1.06-1.el6.x86_64.rpm
    rpm -Uvh sshpass-1.06-1.el6.x86_64.rpm
    yum install sshpass

    echo "서버 수를 입력해 주세요."
    read count

    # HostName 정보 입력
    echo ""
    echo "서버들의 HostName을 입력해주세요."
    echo ""

    x=1
    array=()

    while [ $x -le $count ]
    do
      echo "server$x HostName을 입력해 주세요."
      read hostname
      array+=($hostname)
      echo ""
      x=$(( $x + 1 ))
    done

    # IP 정보 입력
    array2=()
    echo
    echo "서버들의 IP를 입력해주세요."
    echo
    for ((i=0;i<${#array[@]};++i)); do
      echo ${array[i]}" IP 입력해 주세요."
      read ip
      array2+=($ip)
      echo ""
    done

    # 입력 정보 Check
    echo
    for ((i=0;i<${#array[@]};++i)); do
      printf "%s' IP: %s\n" "${array[i]}" "${array2[i]}"
    done

    echo "입력한 정보들이 맞다면 Y(or y)를 눌러주세요."
    read answer
    if [[ $answer != "Y" && $answer != "y" ]]; then
      echo
      echo "스크립트를 다시 실행한 후 정확한 정보를 입력하세요."
      echo
      exit
    fi

    # 계정정보 입력
    echo "서버들의 계정을 입력해 주세요~!"
    read user
    echo
    echo "서버들의 비밀번호를 입력해 주세요!"
    read -s password
    echo

    if [ $password == "" ]; then
      echo
      echo "비밀번호를 입력하지 않으셨습니다. 스크립트를 종료합니다."
      echo
      exit
    fi

    rm -rf $HOME/.ssh

    # 1번 서버 /etc/hosts 파일 설정
    echo '127.0.0.1    localhost' > /etc/hosts
    for index in ${!array[*]}; do 
      echo "${array2[$index]}    ${array[$index]}" >> /etc/hosts
    done

    for index in ${!array[*]}; do
      sshpass -p $password ssh -o StrictHostKeyChecking=no ${array[$index]} "$(typeset -f); genSshKeyScript"
    done

    # 공개 Key 복사
    for index in ${!array[*]}; do
      sshpass -p $password scp -o StrictHostKeyChecking=no $user@${array[$index]}:.ssh/id_rsa.pub $HOME/id_rsa_${array[$index]}.pub
    done

    for index in ${!array[*]}; do
      cat $HOME/id_rsa_${array[$index]}.pub >> $HOME/.ssh/authorized_keys
    done

    for index in ${!array[*]}; do
      sshpass -p $password scp -o StrictHostKeyChecking=no .ssh/authorized_keys $user@${array[$index]}:.ssh/authorized_keys
    done

    rm -f $HOME/id_rsa*


    # Agent Script 실행
    for index in ${!array[*]}; do
      echo "##### agentScript 시작 #####"
      echo ${array[$index]}
      ssh ${array[$index]} "hostname ${array[$index]}"
      ssh ${array[$index]} "sysctl kernel.hostname=${array[$index]}"
      
      if [ $index != 0 ]; then
        ssh ${array[$index]} "rm /etc/hosts"
        scp /etc/hosts $user@${array[$index]}:/etc/hosts
      fi
      ssh ${array[$index]} "$(typeset -f); agentScript"

      echo "##### agentScript 종료 #####"
    done


    # Manager Script 실행
    scp my.cnf $user@${array[0]}:/etc/my.cnf
    ssh ${array[0]} "$(typeset -f); managerScript"
    ```

- mysql 설정 파일 (`cm_install.sh`)
    ```bash
    [mysqld]
    datadir=/var/lib/mysql
    socket=/var/lib/mysql/mysql.sock
    transaction-isolation = READ-COMMITTED
    # Disabling symbolic-links is recommended to prevent assorted security risks;
    # to do so, uncomment this line:
    symbolic-links = 0

    key_buffer_size = 32M
    max_allowed_packet = 32M
    thread_stack = 256K
    thread_cache_size = 64
    query_cache_limit = 8M
    query_cache_size = 64M
    query_cache_type = 1

    max_connections = 550
    #expire_logs_days = 10
    #max_binlog_size = 100M

    #log_bin should be on a disk with enough free space.
    #Replace '/var/lib/mysql/mysql_binary_log' with an appropriate path for your
    #system and chown the specified folder to the mysql user.
    log_bin=/var/lib/mysql/mysql_binary_log

    #In later versions of MySQL, if you enable the binary log and do not set
    #a server_id, MySQL will not start. The server_id must be unique within
    #the replicating group.
    server_id=1

    binlog_format = mixed

    read_buffer_size = 2M
    read_rnd_buffer_size = 16M
    sort_buffer_size = 8M
    join_buffer_size = 8M

    # InnoDB settings
    innodb_file_per_table = 1
    innodb_flush_log_at_trx_commit  = 2
    innodb_log_buffer_size = 64M
    innodb_buffer_pool_size = 4G
    innodb_thread_concurrency = 8
    innodb_flush_method = O_DIRECT
    innodb_log_file_size = 512M

    [mysqld_safe]
    log-error=/var/log/mysqld.log
    pid-file=/var/run/mysqld/mysqld.pid

    sql_mode=STRICT_ALL_TABLES
    ```

    ## 참고자료
    [Cloudera Installation Guide | 5.15.x | Cloudera Documentation](https://www.cloudera.com/documentation/enterprise/5-15-x/topics/installation.html)  
    [Install and Configure MySQL for Cloudera Software | 5.15.x | Cloudera Documentation](https://www.cloudera.com/documentation/enterprise/5-15-x/topics/cm_ig_mysql.html#cmig_topic_5_5_2)  
    [Doing the Install: Cloudera CDH5.12 on CentOS7.3](http://www.alephant.co.uk/Installing_CDH_on_Hadoop-3-Install)