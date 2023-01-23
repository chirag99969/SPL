## Note: Using Public Dataset to come up with SPL (index and sourcetype will be different in enterprise env)

## Root Login Detection
```
source="aws_cloudtrail.zip:*" userIdentity.type=Root eventName=ConsoleLogin
```

## IAM Login with no MFA
```
source="aws_cloudtrail.zip:*"  eventName=ConsoleLogin "additionalEventData.MFAUsed"!=Yes "userIdentity.type"=IAMUser
| dedup userIdentity.arn sourceIPAddress
| table "userIdentity.accountId" "userIdentity.arn" sourceIPAddress "responseElements.ConsoleLogin"
```
![Screenshot_2022-12-22_at_6.30.49_PM](/uploads/3892d02c087a6ff731eb2b9593aeef4b/Screenshot_2022-12-22_at_6.30.49_PM.png)

### This is that same query but including the City and Country of the login:
```
source="aws_cloudtrail.zip:*" ConsoleLogin "additionalEventData.MFAUsed"!=Yes "userIdentity.type"=IAMUser
| dedup userIdentity.arn sourceIPAddress
| iplocation sourceIPAddress
| table "userIdentity.accountId" "userIdentity.arn" sourceIPAddress,
  City, Country "responseElements.ConsoleLogin"
```
![Screenshot_2022-12-22_at_6.29.55_PM](/uploads/4d86721dbb7e9af529a777b1c04fbfd7/Screenshot_2022-12-22_at_6.29.55_PM.png)

## Unauthorized Calls
```
source="aws_cloudtrail.zip:*"  errorCode="AccessDenied" OR errorCode="UnauthorizedOperation"
| stats count by eventName userIdentity.arn
```
![Screenshot_2022-12-22_at_6.32.27_PM](/uploads/bf513e64419bed57c759b4c98411da4a/Screenshot_2022-12-22_at_6.32.27_PM.png)

## Open Security Groups
```
source="aws_cloudtrail.zip:*" eventName = AuthorizeSecurityGroupIngress
"requestParameters.ipPermissions.items{}.ipRanges.items{}.cidrIp"="0.0.0.0/0"
"requestParameters.ipPermissions.items{}.fromPort"=22
OR "requestParameters.ipPermissions.items{}.fromPort"=3389
| stats count values(requestParameters.ipPermissions.items{}.fromPort) as fromPort by userIdentity.arn
```
![Screenshot_2022-12-22_at_6.38.29_PM](/uploads/8cd058569f53d9d58f342144b7ca5fb5/Screenshot_2022-12-22_at_6.38.29_PM.png)

## Hunting for permanent key creation
```
source="aws_cloudtrail.zip:*" eventName=CreateAccessKey userIdentity.type=IAMUser 
| stats count by sourceIPAddress userIdentity.arn responseElements.accessKey.accessKeyId responseElements.accessKey.status responseElements.accessKey.createDate
```
![Screenshot_2022-12-22_at_9.58.17_PM](/uploads/65c5323160e65ccd0c4556296b5365b2/Screenshot_2022-12-22_at_9.58.17_PM.png)
