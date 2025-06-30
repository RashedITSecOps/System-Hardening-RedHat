# System-and-Security-Hardening-RedHat/AlmaLinux/RockyLinux/CentOS
### Add system users like : user1, user2, user3, ....
```shell
mkdir /home/<your_user_name>/.ssh
vim /home/<your_user_name>/.ssh/authorized_keys)
vim /home/<your_user_name>/.ssh/authorized_keys
```
Generate ssh-keygen in the user's workstation 
```shell
ssh-keygen -t rsa -b 2048 
```
Copy and paste the id_rsa.pub file Key content and save into the new system.
```shell
nano /home/<your_user_name>/.ssh/authorized_keys 
```

