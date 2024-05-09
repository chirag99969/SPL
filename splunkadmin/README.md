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
