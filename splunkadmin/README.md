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

# OLD DATA 

```
chirag@cloudshell:~ (my-gcp-project-id)$ gcloud compute instances detach-disk secops-demo1 --disk=secops-demo1
Did you mean zone [asia-southeast1-b] for instance: [secops-demo1] (Y/n)?  n

No zone specified. Using zone [us-central1-c] for instance: [secops-demo1].
ERROR: (gcloud.compute.instances.detach-disk) Could not fetch resource:
 - Invalid resource usage: 'To detach the boot disk, the instance must be in TERMINATED state.'.

chirag@cloudshell:~ (my-gcp-project-id)$ gcloud compute instances detach-disk secops-demo1 --disk=secops-demo1 --zone us-central1-c
ERROR: (gcloud.compute.instances.detach-disk) Could not fetch resource:
 - Invalid resource usage: 'To detach the boot disk, the instance must be in TERMINATED state.'.

chirag@cloudshell:~ (my-gcp-project-id)$ gcloud compute disks snapshot secops-demo1 \
  --snapshot-names=splunk-data-snapshot \
  --zone=us-central1-c
Creating snapshot(s) splunk-data-snapshot...done.                                                                                           
chirag@cloudshell:~ (my-gcp-project-id)$ gcloud compute disks create splunk-data-copy \
  --source-snapshot=splunk-data-snapshot \
  --zone=us-central1-c


^C

Command killed by keyboard interrupt


chirag@cloudshell:~ (my-gcp-project-id)$ gcloud compute disks create splunk-data-copy   --source-snapshot=splunk-data-snapshot   --zone=us-central1-c
ERROR: (gcloud.compute.disks.create) Could not fetch resource:
 - The resource 'projects/my-gcp-project-id/zones/us-central1-c/disks/splunk-data-copy' already exists

chirag@cloudshell:~ (my-gcp-project-id)$ gcloud compute disks create splunk-data-copy1   --source-snapshot=splunk-data-snapshot   --zone=us-central1-c
Created [https://www.googleapis.com/compute/v1/projects/my-gcp-project-id/zones/us-central1-c/disks/splunk-data-copy1].
NAME: splunk-data-copy1
ZONE: us-central1-c
SIZE_GB: 100
TYPE: pd-standard
STATUS: READY
chirag@cloudshell:~ (my-gcp-project-id)$ gcloud compute instances attach-disk secops-demo3 \
  --disk=splunk-data-copy1 \
  --zone=us-central1-c
Updated [https://www.googleapis.com/compute/v1/projects/my-gcp-project-id/zones/us-central1-c/instances/secops-demo3].
chirag@cloudshell:~ (my-gcp-project-id)$

```

### OPTION 1

```
# Create a mount point
sudo mkdir -p /mnt/old-vm-disk

# Mount the main partition from the attached disk (sdb1)
sudo mount /dev/sdb1 /mnt/old-vm-disk

# Verify the Splunk directory is there
ls -la /mnt/old-vm-disk/opt/splunk

# Download and install Splunk
wget -O splunk-latest.tgz "https://download.splunk.com/products/splunk/releases/latest/linux/splunk-latest-linux-x86_64.tgz"
sudo tar -xzf splunk-latest.tgz -C /opt

[volume:old_data]
path = /mnt/old-vm-disk/opt/splunk/var/lib/splunk

[main]
homePath = volume:old_data/defaultdb/db/db
coldPath = volume:old_data/defaultdb/db/colddb
thawedPath = $SPLUNK_DB/defaultdb/thaweddb

```

### OPTION 2
```
# Install Splunk first
wget -O splunk-latest.tgz "https://download.splunk.com/products/splunk/releases/latest/linux/splunk-latest-linux-x86_64.tgz"
sudo tar -xzf splunk-latest.tgz -C /opt

# Stop Splunk if it's running
sudo /opt/splunk/bin/splunk stop

# Copy the indexed data (this preserves permissions)
sudo rsync -avz /mnt/old-vm-disk/opt/splunk/var/lib/splunk/ /opt/splunk/var/lib/splunk/

# Copy configurations if needed
sudo rsync -avz /mnt/old-vm-disk/opt/splunk/etc/ /opt/splunk/etc/

# Set proper ownership
sudo chown -R splunk:splunk /opt/splunk
```
