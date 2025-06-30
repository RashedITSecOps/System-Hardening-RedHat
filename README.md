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
Port xxxx                                  #Use custom port, e.g. 3917
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
```shell
systemctl restart zabbix-agent2
systemctl enable zabbix-agent2
```

### User Command Logging
```shell
# Configure user command execution log, For BASH shells, edit the system-wide BASH runtime config file, append the below line.
nano /etc/bashrc
export PROMPT_COMMAND='RETRN_VAL=$?;logger -p local6.debug "$(whoami) [$$]: $(history 1 | sed "s/^[ ]*[0-9]\+[ ]*//") [$RETRN_VAL]"'

# Set the syslogger to trap local6 to a log file by adding this line 
nano /etc/rsyslog.conf
#User command logging
local6.* /var/log/commands.log

# Add commands.log in logrotate policy
vim /etc/logrotate.d/syslog
/var/log/commands.log 
```

### Configure system status info message (during login)
```shell
# Install the sysstat package, write a new file and add the file location to user’s .bash_profile file to get system info at login time, as below.

dnf -y install sysstat

nano /home/system-info-welcome.sh
#!/bin/bash
S="************************************"
D="-------------------------------------"
#--------Checking the availability of sysstat package..........#
if [ ! -x /usr/bin/mpstat ]
then
printf "\nError : Either \"mpstat\" command not available OR \"sysstat\" package is not properly
installed. Please make sure this package is installed and working properly!, then run this script.\
n\n"
exit 1
fi
echo -e "$S Health Status Report $S"
echo -e "\nOperating System Details"
echo -e "$D"
printf "Hostname :" $(hostname -f > /dev/null 2>&1) && printf " $(hostname -f)" || printf " $
(hostname -s)"
[ -x /usr/bin/lsb_release ] && echo -e "\nOperating System :" $(lsb_release -d|awk -F: '{print
$2}'|sed -e 's/^[ \t]*//') || echo -e "\nOperating System :" $(cat /etc/system-release)
echo -e "Kernel Version :" $(uname -r)
printf "OS Architecture :" $(arch | grep x86_64 2>&1 > /dev/null) && printf " 64 Bit OS\n" ||
printf " 32 Bit OS\n"
#--------Print system uptime-------#
UPTIME=$(uptime)
echo $UPTIME|grep day 2>&1 > /dev/null
if [ $? != 0 ]
then
echo $UPTIME|grep -w min 2>&1 > /dev/null && echo -e "System Uptime : "$(echo $UPTIME|awk '{print
$2" by "$3}'|sed -e 's/,.*//g')" minutes" || echo -e "System Uptime : "$(echo $UPTIME|awk '{print
$2" by "$3" "$4}'|sed -e 's/,.*//g')" hours"
else
echo -e "System Uptime :" $(echo $UPTIME|awk '{print $2" by "$3" "$4" "$5" hours"}'|sed -e 's/,//g')
fi
echo -e "Current System Date & Time : "$(date +%c)
#--------Check for any zombie processes--------#
echo -e "\n\nChecking For Zombie Processes"
echo -e "$D"
ps -eo stat|grep -w Z 1>&2 > /dev/null
if [ $? == 0 ]
then
echo -e "Number of zombie process on the system are :" $(ps -eo stat|grep -w Z|wc -l)
echo -e "\n Details of each zombie processes found
"
echo -e " $D"
ZPROC=$(ps -eo stat,pid|grep -w Z|awk '{print $2}')
for i in $(echo "$ZPROC")
do
ps -o pid,ppid,user,stat,args -p $i
done
else
echo -e "No zombie processes found on the system."
fi
#--------Check for RAM Utilization--------#
MEM_DETAILS=$(cat /proc/meminfo)
echo -e "\n\nChecking Memory Usage Details"
echo -e "$D"
echo -e "Total RAM : "$(echo "$MEM_DETAILS"|grep MemTotal|awk '{print $2/1024/1024}') "GB"
echo -e "Used RAM : "$(free -m|grep -w Mem:|awk '{print $3/1024}') "GB"
echo -e "Free RAM : "$(echo "$MEM_DETAILS"|grep -w MemFree |awk '{print $2/1024/1024}') "GB"

#--------Check for Processor Utilization (current data)--------#
#echo -e "\n\nChecking For Processor Utilization"
echo -e "$D"
#echo -e "Manufacturer: "$(dmidecode -s processor-manufacturer|uniq)
#echo -e "Processor Model: "$(dmidecode -s processor-version|uniq)
#if [ -e /usr/bin/lscpu ]
#then
#{
#echo -e "No. Of Processor(s) :" $(lscpu|grep -w "Socket(s):"|awk -F: '{print $2}')
#echo -e "No. of Core(s) per processor :" $(lscpu|grep -w "Core(s) per socket:"|awk -F: '{print #$2}')
#}
#else
#{
#echo -e "No. Of Processor(s) Found :" $(grep -c processor /proc/cpuinfo)
#echo -e "No. of Core(s) per processor :" $(grep "cpu cores" /proc/cpuinfo|uniq|wc -l)
#}
#fi
echo -e "\nCurrent Processor Utilization Summary :\n"
mpstat|tail -2
```

### Allow required IPs and Ports (for SSH custom port, Zabbix-agent2 etc.) into FirewallD [For VPS/On-prem-VM] 
```shell
firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.100" port port="3917" protocol="tcp" accept'
firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="192.168.1.100" port port="10050-10051" protocol="tcp" accept'
firewall-cmd --reload
firewall-cmd --list-all
```
### OR
### Allow required IPs and Ports (for SSH custom port, Zabbix-agent2 etc.) into Security Group [For AWS VM]  
