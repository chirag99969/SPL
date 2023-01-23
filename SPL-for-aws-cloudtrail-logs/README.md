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
<img width="1789" alt="image" src="https://user-images.githubusercontent.com/69359027/214099213-353e9d06-2bba-415f-b40f-ea33837e604f.png">


### This is that same query but including the City and Country of the login:
```
source="aws_cloudtrail.zip:*" ConsoleLogin "additionalEventData.MFAUsed"!=Yes "userIdentity.type"=IAMUser
| dedup userIdentity.arn sourceIPAddress
| iplocation sourceIPAddress
| table "userIdentity.accountId" "userIdentity.arn" sourceIPAddress,
  City, Country "responseElements.ConsoleLogin"
```
<img width="1788" alt="image" src="https://user-images.githubusercontent.com/69359027/214099531-0ec9c86b-8623-4c0e-a82c-8963cb0c8f9e.png">


## Unauthorized Calls
```
source="aws_cloudtrail.zip:*"  errorCode="AccessDenied" OR errorCode="UnauthorizedOperation"
| stats count by eventName userIdentity.arn
```
<img width="1791" alt="image" src="https://user-images.githubusercontent.com/69359027/214099746-250b0ef2-270f-41c4-9a8d-056636ccd740.png">


## Open Security Groups
```
source="aws_cloudtrail.zip:*" eventName = AuthorizeSecurityGroupIngress
"requestParameters.ipPermissions.items{}.ipRanges.items{}.cidrIp"="0.0.0.0/0"
"requestParameters.ipPermissions.items{}.fromPort"=22
OR "requestParameters.ipPermissions.items{}.fromPort"=3389
| stats count values(requestParameters.ipPermissions.items{}.fromPort) as fromPort by userIdentity.arn
```
<img width="1792" alt="image" src="https://user-images.githubusercontent.com/69359027/214099914-bc40235a-62c3-4964-b9b6-b8282ca67f00.png">


## Hunting for permanent key creation
```
source="aws_cloudtrail.zip:*" eventName=CreateAccessKey userIdentity.type=IAMUser 
| stats count by sourceIPAddress userIdentity.arn responseElements.accessKey.accessKeyId responseElements.accessKey.status responseElements.accessKey.createDate
```
<img width="1792" alt="image" src="https://user-images.githubusercontent.com/69359027/214100049-a35d1ad2-a625-4e26-a9e9-80e9ab4c6ae1.png">

