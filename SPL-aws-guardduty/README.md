

# SPL for guardduty findings

## Anomalous activities/findings by type and region. Sort by count. 
```
sourcetype="aws:cloudwatch:guardduty*" 
| stats count by Region Type 
| sort -count
```


## Root User Activities
```
sourcetype="aws:cloudwatch:guardduty*" Type="Policy:IAMUser/RootCredentialUsage" 
| stats count values(Service.Action.AwsApiCallAction.RemoteIpDetails.*) as * by AccountId Region Description _time
```


## BruteForce Activities by potential malicious actors
```
sourcetype="aws:cloudwatch:guardduty*" Type="*SSHBruteForce" 
| rename Resource.InstanceDetails.InstanceId as InstanceId
| stats count values(Resource.InstanceDetails.NetworkInterfaces{}.PrivateIpAddress) as PrivateIp values(Resource.InstanceDetails.NetworkInterfaces{}.PublicIp) as PublicIp values(Service.Action.NetworkConnectionAction.LocalPortDetails.*) as * values(Service.Action.NetworkConnectionAction.RemoteIpDetails.IpAddressV4) as ThreatIp values(Service.Action.NetworkConnectionAction.RemoteIpDetails.City.CityName) as City values(Service.Action.NetworkConnectionAction.RemoteIpDetails.Country.CountryName) as country values(Service.Action.NetworkConnectionAction.RemoteIpDetails.Organization.*) as * by InstanceId AccountId Region 
| sort country
```


## BruteForce Activities by potential malicious actors from specific countries
```
sourcetype="aws:cloudwatch:guardduty*" Type="*SSHBruteForce" 
| rename Resource.InstanceDetails.InstanceId as InstanceId Service.Action.NetworkConnectionAction.RemoteIpDetails.Country.CountryName as country
| search country=China OR country=Russia
| stats count values(Resource.InstanceDetails.NetworkInterfaces{}.PrivateIpAddress) as PrivateIp values(Resource.InstanceDetails.NetworkInterfaces{}.PublicIp) as PublicIp values(Service.Action.NetworkConnectionAction.LocalPortDetails.*) as * values(Service.Action.NetworkConnectionAction.RemoteIpDetails.IpAddressV4) as ThreatIp values(Service.Action.NetworkConnectionAction.RemoteIpDetails.City.CityName) as City values(country) values(Service.Action.NetworkConnectionAction.RemoteIpDetails.Organization.*) as * by InstanceId AccountId Region 
| sort country
```


## DGAs Domains Being Queried by Ec2 Instances
```
sourcetype="aws:cloudwatch:guardduty" Type="Trojan:EC2/DGADomainRequest.B"
| stats count values(Resource.InstanceDetails.InstanceId) as IntanceId values(Resource.InstanceDetails.NetworkInterfaces{}.PublicIp) as PublicIp values(Resource.InstanceDetails.NetworkInterfaces{}.SecurityGroups{}.GroupId) as sgIds values(Resource.InstanceDetails.NetworkInterfaces{}.SubnetId) as subnets values(Resource.InstanceDetails.NetworkInterfaces{}.VpcId) as vpcIds values(Service.Action.DnsRequestAction.Domain) as domain by _time Region
```


## C&C Domain being Queried by Ec2 
```
sourcetype="aws:cloudwatch:guardduty" Type="Backdoor:EC2/C&CActivity.B!DNS" 
| stats count values(Resource.InstanceDetails.InstanceId) as IntanceId values(Resource.InstanceDetails.NetworkInterfaces{}.PublicIp) as PublicIp values(Resource.InstanceDetails.NetworkInterfaces{}.SecurityGroups{}.GroupId) as sgIds values(Resource.InstanceDetails.NetworkInterfaces{}.SubnetId) as subnets values(Resource.InstanceDetails.NetworkInterfaces{}.VpcId) as vpcIds values(Service.Action.DnsRequestAction.Domain) as domain by _time Region
```


## Defense Evasion Cloudtrail logs disabled
```
sourcetype="aws:cloudwatch:guardduty" Type="Stealth:IAMUser/CloudTrailLoggingDisabled" 
| stats count values(Service.Action.AwsApiCallAction.RemoteIpDetails.Organization.Org) as ISP values(Service.Action.AwsApiCallAction.RemoteIpDetails.IpAddressV4) as IP values(Service.Action.AwsApiCallAction.RemoteIpDetails.Country.CountryName) as country values(Service.Action.AwsApiCallAction.AffectedResources.AWS::CloudTrail::Trail) as TrailName by Resource.AccessKeyDetails.UserName _time Region
```


## Port Scanning by Ec2 Instances
```
sourcetype="aws:cloudwatch:guardduty" Type="Recon:EC2/Portscan"
| stats count values(Resource.InstanceDetails.InstanceId) as IntanceId values(Resource.InstanceDetails.NetworkInterfaces{}.PublicIp) as PublicIp values(Resource.InstanceDetails.NetworkInterfaces{}.SecurityGroups{}.GroupId) as sgIds values(Resource.InstanceDetails.NetworkInterfaces{}.SubnetId) as subnets values(Resource.InstanceDetails.NetworkInterfaces{}.VpcId) as vpcIds values(Service.AdditionalInfo.Value) as PortsScanned values(Service.Action.NetworkConnectionAction.RemoteIpDetails.IpAddressV4) as RemoteIps values(Service.Action.NetworkConnectionAction.RemoteIpDetails.Country.CountryName) as Country values(Service.Action.NetworkConnectionAction.RemoteIpDetails.City.CityName) as City by _time Region
```


## Privilege Escalation, Root Containers launched
```
sourcetype="aws:cloudwatch:guardduty" Type="PrivilegeEscalation:Kubernetes/PrivilegedContainer"
| rename Resource.KubernetesDetails.KubernetesWorkloadDetails.Containers{}.Name as containerName
| stats count values(Resource.KubernetesDetails.KubernetesWorkloadDetails.Type) as type values(Resource.KubernetesDetails.KubernetesWorkloadDetails.Namespace) as namespace  values(Resource.KubernetesDetails.KubernetesWorkloadDetails.Containers{}.Image) as Image values(Resource.KubernetesDetails.KubernetesUserDetails.Username) as username values(Service.Action.KubernetesApiCallAction.Verb) as verb values(Resource.EksClusterDetails.Arn) as EksArn by _time containerName
```


## Ec2 Querying IPs associated with Tor Nodes
```
sourcetype="aws:cloudwatch:guardduty" Type="UnauthorizedAccess:EC2/TorClient"
| stats count values(Resource.InstanceDetails.InstanceId) as IntanceId values(Resource.InstanceDetails.NetworkInterfaces{}.PublicIp) as PublicIp values(Resource.InstanceDetails.NetworkInterfaces{}.SecurityGroups{}.GroupId) as sgIds values(Resource.InstanceDetails.NetworkInterfaces{}.SubnetId) as subnets values(Resource.InstanceDetails.NetworkInterfaces{}.VpcId) as vpcIds values(Service.Action.NetworkConnectionAction.RemoteIpDetails.City.CityName) as city values(Service.Action.NetworkConnectionAction.RemoteIpDetails.Country.CountryName) as country values(Service.Action.NetworkConnectionAction.RemoteIpDetails.IpAddressV4) as remoteIps values(Service.Action.NetworkConnectionAction.RemoteIpDetails.Organization.AsnOrg) as ORGs values(Service.Action.NetworkConnectionAction.RemotePortDetails.Port) as PortNo values(Service.Action.NetworkConnectionAction.RemotePortDetails.PortName) as Portname by _time Region
```



## Kubernetes Successful Anonymous Access 
```
sourcetype="aws:cloudwatch:guardduty" Type="Discovery:Kubernetes/SuccessfulAnonymousAccess"
| stats count values(Resource.EksClusterDetails.Name) as Eksclustername values(Resource.EksClusterDetails.VpcId) as vpcId values(Resource.KubernetesDetails.KubernetesUserDetails.Username) as userName values(Resource.KubernetesDetails.KubernetesUserDetails.Groups{}) as groups values(Resource.EksClusterDetails.Status) as status values(Service.Action.KubernetesApiCallAction.RemoteIpDetails.Organization.Org) as ORG values(Service.Action.KubernetesApiCallAction.RemoteIpDetails.IpAddressV4) as AttackerIps values(Service.Action.KubernetesApiCallAction.RemoteIpDetails.Country.CountryName) as country values(Service.Action.KubernetesApiCallAction.StatusCode) as statuscode values(Service.Action.KubernetesApiCallAction.Verb) as verb by _time Region Description
```

