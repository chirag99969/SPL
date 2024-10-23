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


## props.conf and transforms.conf

##### props.conf
```
[aws:s3:eks:kube-apiserver-audit]
KV_MODE = json
LINE_BREAKER = ([\r\n]+)
NO_BINARY_CHECK = true
category = Structured
description = AWS Canary JSON Summary File
disabled = false
pulldown_type = true

[aws:s3:eks:authenticator]
DATETIME_CONFIG = 
LINE_BREAKER = ([\r\n]+)
MAX_TIMESTAMP_LOOKAHEAD = 50
NO_BINARY_CHECK = true
SHOULD_LINEMERGE = false
TIME_PREFIX = time=\s
TIME_FORMAT = %Y-%m-%dT%H:%M:%SZ
category = Application
description = AWS Canary Detailed TXT file
disabled = false
pulldown_type = true


[aws:s3:eks]
TRANSFORMS-aws_eks = set_aws_json, set_aws_auth, drop_aws_kubeapiserver, drop_aws_kube_controller, drop_aws_cloud_controller, drop_aws_kube_scheduler
```

##### transforms.conf

```
[set_aws_json]
SOURCE_KEY = MetaData:Source
#REGEX = ^s3://bucketName/gc02-ore/*/*/*/*-*/kube-apiserver-audit-*
REGEX = source="s3://([a-z-0-9]+)/([a-z0-9_\.-]+)/\d{4}/\d{2}/\d{2}/\d{2}-\d{2}/([a-z0-9-]+)/kube-apiserver-audit-*"
FORMAT = sourcetype::aws:se:eks:kube-apiserver-audit
DEST_KEY = MetaData:Sourcetype

[set_aws_auth]
SOURCE_KEY = MetaData:Source
#REGEX = source=s3:\/\/bucketName\/gc02-ore\/*\/*\/*\/*-*\/authenticator-*
REGEX = source="s3://([a-z-0-9]+)/([a-z0-9_\.-]+)/\d{4}/\d{2}/\d{2}/\d{2}-\d{2}/([a-z0-9-]+)/authenticator-*"
FORMAT = sourcetype::aws:s3:eks:authenticator
DEST_KEY = MetaData:Sourcetype

[drop_aws_kubeapiserver]
SOURCE_KEY = MetaData:Source
#REGEX = source=s3:\/\/bucketName\/gc02-ore\/*\/*\/*\/*-*\/kube-apiserver-*
REGEX = source="s3://([a-z-0-9]+)/([a-z0-9_\.-]+)/\d{4}/\d{2}/\d{2}/\d{2}-\d{2}/([a-z0-9-]+)/kube-apiserver-*"
FORMAT = nullQueue
DEST_KEY = queue

[drop_aws_kube_controller]
SOURCE_KEY = MetaData:Source
#REGEX = ^s3://bucketName/gc02-ore/*/*/*/*-*/kube-controller-manager-*
REGEX = source="s3://([a-z-0-9]+)/([a-z0-9_\.-]+)/\d{4}/\d{2}/\d{2}/\d{2}-\d{2}/([a-z0-9-]+)/kube-controller-manager-*"
FORMAT = nullQueue
DEST_KEY = queue

[drop_aws_cloud_controller]
SOURCE_KEY = MetaData:Source
#REGEX = ^s3://bucketName/gc02-ore/*/*/*/*-*/cloud-controller-manager-*
REGEX = source="s3://([a-z-0-9]+)/([a-z0-9_\.-]+)/\d{4}/\d{2}/\d{2}/\d{2}-\d{2}/([a-z0-9-]+)/cloud-controller-manager-*"
FORMAT = nullQueue
DEST_KEY = queue

[drop_aws_kube_scheduler]
SOURCE_KEY = MetaData:Source
#REGEX = ^s3://bucketName/gc02-ore/*/*/*/*-*/kube-scheduler-*
REGEX = source="s3://([a-z-0-9]+)/([a-z0-9_\.-]+)/\d{4}/\d{2}/\d{2}/\d{2}-\d{2}/([a-z0-9-]+)/kube-scheduler-*"
FORMAT = nullQueue
DEST_KEY = queue
```
# props and transforms WORKING OKAY

## props
```
[aws:s3:route53]
TRANSFORMS-aws_route53 = aws_route53_test
```

## transforms
```
[aws_route53_test]
SOURCE_KEY = MetaData:Source
REGEX = s3:\/\/[a-z1-9-.]+\/AWSLogs\/(\d{12})\/*
FORMAT = sourcetype::aws:route53:test
DEST_KEY = MetaData:Sourcetype
```

# props.conf gor guardduty

* props.conf for sourcetype aws:cloudwatchlogs:guardduty

* Existing configuration/Workable one

```
[aws:cloudwatchlogs:guardduty]
LINE_BREAKER=(([\r\n]+)|(?={"schemaVersion":"[\d.]+","accountId":))
SHOULD_LINEMERGE = false
NO_BINARY_CHECK = false
TRUNCATE = 8388608
TIME_PREFIX = \"updatedAt\"\s*\:\s*\"
TIME_FORMAT = %Y-%m-%dT%H:%M:%S%Z
MAX_TIMESTAMP_LOOKAHEAD = 40
KV_MODE = json
```
