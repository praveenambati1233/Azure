

**Use case :** X client wants to create alerts for application errors that are being logged in virtual machine. So they can proactively notify application teams for mitigation plan.
**Solution :** Fluentd is an open-source data collector for a unified logging layer. Fluentd allows you to unify data collection and consumption for better use and understanding of data. Azure Log Analytics output plugin for Fluentd comes in resue which the plugin aggregates semi-structured data in real-time and writes the buffered data via HTTPS request to Azure Log Analytics.


**Architecture **

Below is the high level diagram for monitoring the custom logs in Log Analytics workspace in Linux OS.


Image  Fluent.image


**Implementation**

Step 1:  Enable LAW extension on the VM 


Image VM_LOGS_1

Image  VM_LOGS_2


Once VM is connected to LAW. DependencyAgentLinux  & OMSAgentForLinux extensions get installed on the VM. 

Image VMEXT 

Step 2:  Install td-agent on Linux VM 

```
echo "=============================="
echo " td-agent Installation Script "
echo "=============================="
echo "This script requires superuser access to install rpm packages."
echo "You will be prompted for your password by sudo."
 
#clear any previous sudo permission
sudo -k
 
#run inside sudo
sudo sh <<SCRIPT
 
#add GPG key
  rpm --import https://packages.treasuredata.com/GPG-KEY-td-agent
 
  #add treasure data repository to yum
  cat >/etc/yum.repos.d/td.repo <<'EOF';
[treasuredata]
name=TreasureData
#baseurl=http://packages.treasuredata.com/4/redhat/$releasever/$basearch
baseurl=https://packages.treasuredata.com/3/redhat/8/x86_64/
gpgcheck=1
gpgkey=https://packages.treasuredata.com/GPG-KEY-td-agent
EOF
 
  #update your sources
  yum check-update
 
  #install the toolbelt
  yes | yum install -y td-agent
 
SCRIPT
 
#message
echo ""
```


Step 2.1: Upgrade the fluentd version 
 
`$sudo td-agent-gem install fluentd --version=1.14.3`
 
Step 2.2: Install pre-requisite utilities 
 
`$sudo yum group install "Development Tools"`
 
Step 2.3: Install fluent-plugin-azure-loganalytics
 
`$ sudo /usr/sbin/td-agent-gem install fluent-plugin-azure-loganalytics`


Step 3: Validate the installation
 
`$ sudo /etc/init.d/td-agent start
$ sudo /etc/init.d/td-agent status
$ sudo systemctl restart td-agent`
 
 
Step 4: create sample script which results some logs at /tmp/mylog.log
 
 
`[root@fluentd-poc-vm td-agent]# cd /tmp
[root@fluentd-poc-vm tmp]# cat > mylog.log
This is sample log for testing VM --> LAW      
^C
[root@fluentd-poc-vm tmp]# chown td-agent:td-agent mylog.log
` 
 
Step 5: Add below configuration at `/etc/td-agent/td-agent `
 
 
Get the workspace id and primary key from LAW - Agents Management 

image agent

Add below configuration at `/etc/td-agent/td-agent.conf`


```
<source>
    @type tail
    path /tmp/mylog.log
    pos_file /tmp/mylog.pos
    format none
    tag azure-loganalytics.access
</source>
 
 
<match azure-loganalytics.**>
    @type azure-loganalytics
    customer_id XXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    shared_key XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    log_type TailLog
</match>

```


Step 6:
 
Start / restart the td-agent to get the latest changes. 
 
`[root@fluentd-poc-vm td-agent]# sudo /etc/init.d/td-agent start
Starting td-agent (via systemctl):  [  OK  ]
` 
 
Step 7:  Ensure there are no errors in td-agent 
 
In td-agent.log you will see log `/tmp/mylog.log `is getting read 
 
`2022-05-09 13:02:49 +0000 [info]: #0 [input_forward] listening port port=24224 bind="0.0.0.0"
2022-05-09 13:02:49 +0000 [info]: #0 following tail of /tmp/mylog.log
2022-05-09 13:02:49 +0000 [info]: #0 fluentd worker is now running worker=0
` 

Position file will be created 
 
`-rw-r--r--. 1 td-agent td-agent 49 May  9 13:02 mylog.pos`
 
 
Step 8:  TailLog_CL will be created in Log Analytics Workspace under Custom Logs.


image LAW

Step 9: Create a alert 


image alert


Sample test alert


image email