# System-and-Security-Hardening-RedHat/AlmaLinux/RockyLinux/CentOS

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
Port xxxx                                  #use custom port, e.g. 3914
LogLevel INFO
PermitRootLogin no
PermitEmptyPasswords no
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication yes
X11Forwarding no
```
