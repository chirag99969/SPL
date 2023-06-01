# Listing sourcetypes

```
|  metadata type=sourcetypes index=botsv1
```
![image](/uploads/c2e5714016fa4728df5c0295411c15d2/image.png)



# 1 EVAL command

### 1.1 UC

```
index=botsv1 sourcetype="stream:tcp"
| eval MB = round(bytes/(1024*1024))
```
![image](/uploads/d17762ad53ca3f28c3cd98a0617c66dc/image.png)

### 1.2 UC 

```
index=botsv1 sourcetype=WinEventLog* EventCode=4624 
| eval Account_Name=upper(Account_Name)
```
* BEFORE
![image](/uploads/babce5a80e10b71a422877e5c9d12bd5/image.png)
* AFTER
![image](/uploads/b348af60b0f7e3bf8465c5d203d72917/image.png)

### 1.3 UC

```
index=botsv1 sourcetype="stream:dns" "query_type{}"=A
| eval queryLen = len(query)
```

### 1.4 UC

```
index=botsv1 sourcetype="fgt_traffic" 
| eval dstcountry = if(dstcountry="United States", "United States", "International")
```

