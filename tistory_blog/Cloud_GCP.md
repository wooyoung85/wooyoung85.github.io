

```bash
#!/bin/bash
sudo yum -y install expect

expect -c "set timeout 10
spawn sudo passwd
expect \"New password:\"
send \"gcp123!@#\r\"
expect \"Retype new password:\"
send \"gcp123!@#\r\"
expect eof"

sudo sed -i 's/^PermitRootLogin .*/PermitRootLogin yes/' /etc/ssh/sshd_config
sudo sed -i "s/^PasswordAuthentication .*/PasswordAuthentication yes/g" /etc/ssh/sshd_config

sudo service sshd restart
```


```bash
gcloud compute instances create node2 node3 node4 node5 \
--zone=asia-northeast1-a \
--machine-type=n1-standard-1 \
--image-family=centos-6 \
--image-project=centos-cloud  \
--boot-disk-size=100GB \
--metadata-from-file startup-script=install.sh
```

```bash
gcloud compute instances create node1 \
--zone=asia-northeast1-a \
--custom-cpu=2 \
--custom-memory=10 \
--image-family=centos-6 \
--image-project=centos-cloud  \
--boot-disk-size=100GB \
--metadata-from-file startup-script=install.sh
```