# System-and-Security-Hardening-RedHat/AlmaLinux/RockyLinux/CentOS

### Change System Hostname
```shell
hostnamectl set-hostname xxxxxxxxxxxx
```
### Add system users like : user1, user2, user3, ....
```shell
mkdir /home/<user1>/.ssh
vim /home/<your_user_name>/.ssh/authorized_keys)
vim /home/<your_user_name>/.ssh/authorized_keys
```
Create an SSH Key Pair for user1 (in user1's workstation). 
```shell
ssh-keygen -t rsa -b 2048 
```
Store user's id_rsa.pub key file content into the new system and cofirm user's password revocation
```shell
nano /home/<your_user_name>/.ssh/authorized_keys
passwd -d <user1>
```
Provide sudo privilege to the newly created user
```shell
visuo
user1   ALL=(ALL:ALL) ALL
```

### Configure the SSH service for secure remote login
```shell
nano /etc/ssh/sshd_config
Port xxxx                                  #Use custom port, e.g. 3914
IgnoreRhosts yes                           #Ignore rhosts/shosts trusted based authentication 
LogLevel INFO
PermitRootLogin no
PermitEmptyPasswords no
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication yes       #Enables PAM/OTP/etc.auth mechanisms 
X11Forwarding no                          #Disables GUI app forwarding
AllowUsers user1 user2 user3
```

### Install Zabbix-agent to the new Server
a. Install Zabbix repository
```shell
nano /etc/yum.repos.d/epel.repo
[epel]
...
excludepkgs=zabbix*
```
Proceed with installing zabbix repository.
```shell
rpm -Uvh https://repo.zabbix.com/zabbix/7.2/release/rhel/9/noarch/zabbix-release-latest-7.2.el9.noarch.rpm
dnf clean all
```
b. Install Zabbix agent2
```shell
dnf install zabbix-agent2
```
e. Configure zabbix-agent2.conf
```shell
nano /etc/zabbix/zabbix_agent2.conf
Server=[zabbix-server-ip]
ServerActive=[zabbix-server-ip]
Hostname=[new-system-hostname]
```
f. Start Zabbix agent2 process
```
systemctl restart zabbix-agent2
systemctl enable zabbix-agent2
```
