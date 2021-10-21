# Fail2ban
Fail2ban for Centos7  


## Installing Fail2Ban  
To install Fail2Ban on CentOS 7, we will have to install EPEL (Extra Packages for Enterprise Linux) repository first. EPEL contains additional packages for all CentOS versions, one of these additional packages is Fail2Ban.  
The following commands must be executed after switching to the root user.  
```  
yum -y install epel-release  
yum -y install fail2ban fail2ban-systemd  
```  
If you have SELinux installed, then update the SELinux policies:  
`yum update -y selinux-policy*`  

## Configure settings for Fail2Ban  

Once installed, we will have to configure and customize the software with a jail.local configuration file. The jail.local file overrides the jail.conf file and is used to make your custom configuration update safe.  

* Make a copy of the jail.conf file and save it with the name jail.local:  
`cp -pf /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`  
Open the jail.local file for editing in Nano with the following command.  
`vi /etc/fail2ban/jail.local`  
The file code may consist of many lines of codes which execute to prevent a ban on one or many IP addresses, set bantime duration, etc. A typical jail configuration file contains the following lines.  

```  
[DEFAULT]  

#  
# MISCELLANEOUS OPTIONS
#  

# "ignoreip" can be an IP address, a CIDR mask or a DNS host. Fail2ban will not
# ban a host which matches an address in this list. Several addresses can be
# defined using space separator.
ignoreip = 127.0.0.1/8

# External command that will take an tagged arguments to ignore, e.g. <ip>,  
# and return true if the IP is to be ignored. False otherwise.  
#  
# ignorecommand = /path/to/command <ip>  
ignorecommand =  

# "bantime" is the number of seconds that a host is banned.  
bantime = 864000

# A host is banned if it has generated "maxretry" during the last "findtime"  
# seconds.  
findtime = 7200

# "maxretry" is the number of failures before a host get banned.  
maxretry = 10
```  


Ignoreip is used to set the list of IPs which will not be banned. The list of IP addresses should be given with a space separator. This parameter is used to set your personal IP address (if you access the server from a fixed IP).  
Bantime parameter is used to set the duration of seconds for which a host needs to be banned.  
Findtime is the parameter which is used to check if a host must be banned or not. When the host generates maxrety in its last findtime, it is banned.  
Maxretry is the parameter used to set the limit for the number of retry's by a host, upon exceeding this limit, the host is banned.  


##  Add a jail file to protect SSH.  
* Create a new file with the Nano editor  
`vi /etc/fail2ban/jail.d/sshd.local`  
To the above file, add the following lines of code.  
```  
[sshd]
enabled = true
port = ssh
#action = firewallcmd-ipset
logpath = %(sshd_log)s
maxretry = 10
bantime = 864000
```  

Parameter enabled is set to true, in order to provide protection, to deactivate protection, it is set to false. The filter parameter checks the sshd configuration file, located in the path /etc/fail2ban/filter.d/sshd.conf.  
The parameter action is used to derive the IP address which needs to be banned using the filter available from /etc/fail2ban/action.d/firewallcmd-ipset.conf.  
Port parameter may be changed to a new value such as port=1212, as is the case. When using port 22, there is no need to change this parameter.  
Logpath provides the path where the log file is stored. This log file is scanned by Fail2Ban.  
Maxretry is used to set the maximum limit for failed login entries.  
Bantime parameter is used to set the duration of seconds for which a host needs to be banned.  

## Running Fail2Ban service  
When you are not running the CentOS Firewall yet, then start it:  
```  
systemctl enable firewalld
systemctl start firewalld
```  

Execute the following lines of command to run the protective Fail2Ban software on the server.  
```  
systemctl enable fail2ban  
systemctl start fail2ban  
```  

## Tracking Failed login entries  
The following command is used to check whether there had been failed attempts to login to sever via ssh port.  
`cat /var/log/secure | grep 'Failed password'`  
Executing above command will get a list of failed root password attempts from different IP addresses. The format of results will be similar to the one showed below:  
```  
Fer 8 12:41:12 htf sshd[5487]: Failed password for root from 108.61.157.25 port 23021 ssh2  
Fer 8 12:41:15 htf sshd[1254]: Failed password for root from 108.61.157.25 port 15486 ssh2  
Fer 8 12:41:16 htf sshd[1254]: Failed password for root from 108.61.157.25 port 24457 ssh2  
Fer 8 12:41:18 htf sshd[1254]: Failed password for root from 108.61.157.25 port 24457 ssh2  
```  
 
##  Checking the banned IPs by Fail2Ban  
The following command is used to get a list of banned IP addresses which were recognized as brute force threats.  
`iptables -L -n`  
* Check the Fal2Ban Status  
Use the following command to check the status of the Fail2Ban jails:  
`fail2ban-client status`  
The result should be similar to this:  
```  
[root@htf ]# fail2ban-client status  
Status  
|- Number of jail: 1  
`- Jail list: sshd  
Unbanning an IP address  
In order to remove an IP address from the banned list, parameter IPADDRESS is set to appropriate IP which needs unbanning. The name "sshd" is the name of the jail, in this case the "sshd" jail that we configured above. The following command does the job.  
fail2ban-client set sshd unbanip IPADDRESS  
```  


