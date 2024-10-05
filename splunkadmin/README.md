# Install Splunk 

```
wget -O splunk-9.0.3-dd0128b1f8cd-linux-2.6-amd64.deb "https://download.splunk.com/products/splunk/releases/9.0.3/linux/splunk-9.0.3-dd0128b1f8cd-linux-2.6-amd64.deb"

mv splunk-9.0.3-dd0128b1f8cd-linux-2.6-amd64.deb /tmp

cd /tmp

cd /tmp

sudo dpkg -i splunk-9.0.3-dd0128b1f8cd-linux-2.6-amd64.deb 

sudo /opt/splunk/bin/splunk enable boot-start --accept-license --answer-yes

sudo service splunk start

```


# iptables

```
# Install iptables
sudo apt-get update
sudo apt-get install iptables

# list Rules
iptables -L -v

# Commands to Block
 sudo iptables -A OUTPUT -d 3.208.215.234 -j REJECT
 sudo iptables -A OUTPUT -d 3.223.109.193 -j REJECT
 sudo iptables -A OUTPUT -d 3.225.203.212 -j REJECT

# To make it persist
sudo apt-get install iptables-persistent
sudo netfilter-persistent save

```

# Splunk admin REST Queries

```
| rest /services/saved/searches 
| search is_scheduled=1 realtime_schedule=1
| search disabled=0
| table title disabled realtime_schedule is_scheduled
```

```
| rest /servicesNS/-/-/saved/searches  | search is_scheduled=1 
| search disabled=0  | search title="ESCU*"
|  table title,  cron_schedule next_scheduled_time eai:acl.owner  actions eai:acl.app action.email action.email.to dispatch.earliest_time dispatch.latest_time search
```

# Generating events or making events 
```
| makeresults count=1 | eval src="45.128.96.194", result="Malicious" | collect index="ioc" sourcetype="sample"
```


# Listing all sourcetype for an index

```
| metadata type=sourcetypes index=node_security
```

# Coorelation Query using subserach

```
index=aws sourcetype=aws:cloudtrail [search index=aws sourcetype=aws:cloudwatchlogs:guardduty
| rename service.action.portProbeAction.portProbeDetails{}.remoteIpDetails.ipAddressV4 as src1
| fields src1
| rename src1 as src]
| table src  _time eventName eventSource userName errorCode aws_account_id
```

# Coorleation Search 

```
sourcetype="aws:s3:sys-sshd"
| rex field=message "\b(?<ip_address>(?:[0-9]{1,3}\.){3}[0-9]{1,3})\b"
| search ip_address!=""
| join type=inner ip_address
    [search sourcetype="aws:cloudwatchlogs:guardduty"
| rename service.action.portProbeAction.portProbeDetails{}.remoteIpDetails.ipAddressV4 as ip_address]
| table _time, ip_address, message
```

#### Data Ingestion per sourcetype in GBs

```
index=_internal source=*metrics.log earliest=-4d | eval GB=kb/(1024*1024) | search group="per_sourcetype_thruput" | timechart span=1d sum(GB) by series limit=40
```

#### Disk Space Utilization for indexers 

```
| rest splunk_server=* /services/server/status/partitions-space | eval usage = round((capacity - free) / 1024, 2) | eval capacity = round(capacity / 1024, 2) | eval compare_usage = usage." / ".capacity | eval pct_usage = round(usage / capacity * 100, 2)  | table updated, splunk_server, mount_point, fs_type, capacity, compare_usage, pct_usage | rename mount_point as "Mount Point", fs_type as "File System Type", compare_usage as "Disk Usage (GB)", capacity as "Capacity (GB)", pct_usage as "Disk Usage (%)" | sort splunk_server
```
