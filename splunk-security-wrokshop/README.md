# Listing sourcetypes

```
|  metadata type=sourcetypes index=botsv1
```
<img width="1750" alt="image" src="https://github.com/chirag99969/SPL/assets/69359027/3f7336b5-ba43-48e9-a550-59c07cfa5d3a">


# 1 EVAL command

### 1.1 UC

```
index=botsv1 sourcetype="stream:tcp"
| eval MB = round(bytes/(1024*1024))
```
<img width="1249" alt="image" src="https://github.com/chirag99969/SPL/assets/69359027/e6c3928a-ed57-4dff-98e9-ee7d4611d2d5">

### 1.2 UC 

```
index=botsv1 sourcetype=WinEventLog* EventCode=4624 
| eval Account_Name=upper(Account_Name)
```
* BEFORE
<img width="1150" alt="image" src="https://github.com/chirag99969/SPL/assets/69359027/7e707f2f-5e0e-4e5c-9e83-ecf6f3e3fd6c">

* AFTER
<img width="1272" alt="image" src="https://github.com/chirag99969/SPL/assets/69359027/4f23779f-29ee-47a4-9dfe-2353c8508dba">

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
<img width="1519" alt="image" src="https://github.com/chirag99969/SPL/assets/69359027/eb9686d5-a397-42c6-a050-b4c023e2c11b">

### 1.5 UC

```
index=botsv1 sourcetype=stream:tcp
| eval byte_size = case(bytes < 500, "Small", bytes < 5000, "Medium", bytes >= 5000, "Large", 1=1, "Unknown")
```
<img width="1217" alt="image" src="https://github.com/chirag99969/SPL/assets/69359027/e4d38ea0-e984-491c-b822-83adc63c67b0">

### 1.6 UC
```
index=botsv1 sourcetype=suricata
| eval month_day=strftime(_time, "%m/%d")
```
<img width="1112" alt="image" src="https://github.com/chirag99969/SPL/assets/69359027/fd1ee2fc-771d-4a26-b65c-a84248e6f709">

### 1.7 Fields (Interesting)
```
index=botsv1 sourcetype=iis
| fields _time, c_ip, s_ip
```
<img width="1080" alt="image" src="https://github.com/chirag99969/SPL/assets/69359027/1cd3c04e-efac-4286-aff4-986bd3076e53">

### 1.8 regex 
```
index=botsv1 sourcetype="wineventlog*" 
| regex Account_Name!="\w+\$"
```
* BEFORE
<img width="1161" alt="image" src="https://github.com/chirag99969/SPL/assets/69359027/bce801a2-3beb-4183-b1fd-c8cdde888422">

* AFTER
<img width="1039" alt="image" src="https://github.com/chirag99969/SPL/assets/69359027/29b1300b-7bea-498e-a5aa-0e0f2442a7cf">

### 1.9 regex

```
index=botsv1 sourcetype=iis cs_Referer=*
| rex field=cs_Referer "http://(?<domain>(\w+\.\w+|\d+\.\d+\.\d+\.\d+))\/"
```
<img width="1207" alt="image" src="https://github.com/chirag99969/SPL/assets/69359027/9eb006b5-6e2f-4300-9bfa-d151e8407765">

# 2 Transforming Commands

### 2.1 table

```
index=botsv1 sourcetype="fgt_traffic"  srccountry!="Reserved" dstport=23
| table _time, srcip, srccountry, dstip, dstport, action, service
```
<img width="1634" alt="image" src="https://github.com/chirag99969/SPL/assets/69359027/9883b2a2-a51b-4538-be5f-406a12c3cdf7">

### 2.2 stats

```
index=botsv1 sourcetype="fgt_traffic"  srccountry!="Reserved" dstport=23
| stats count
```

<img width="873" alt="image" src="https://github.com/chirag99969/SPL/assets/69359027/af5035b2-ed58-45f9-9665-fe1fc3d26d27">

### 2.3 distinct count

```
index=botsv1 sourcetype="fgt_traffic"  srccountry!="Reserved" dstport=23
| stats dc(srccountry) as "Distinct Countries"
```
<img width="760" alt="image" src="https://github.com/chirag99969/SPL/assets/69359027/d96cbdf4-ad6f-4f3c-afe2-4a67999d20bd">

### 2.4 max min count avg

```
index=botsv1 sourcetype="fgt_traffic"  srccountry!="Reserved" src_ip=40.80.148.42
| stats count max(bytes) as max_bytes, min(bytes) as min_bytes, avg(bytes) as avg_bytes
| eval avg_bytes= round(avg_bytes,2)
```

<img width="1755" alt="image" src="https://github.com/chirag99969/SPL/assets/69359027/69528ae6-e570-4ddb-b56f-1962692bba0c">

### 2.4 stats with BY clause

```
index=botsv1 sourcetype=win* EventCode=4624 Account_Name="bob.smith"
| stats count by user, host
```
<img width="1755" alt="image" src="https://github.com/chirag99969/SPL/assets/69359027/2e1c0229-98a3-4fdb-b4b4-f66b3d4a55c2">

### 2.5 stats list function (all occurrences)

```
index=botsv1 sourcetype=win* EventCode=4624 
 | stats list(Account_Name) as users
 ```
 <img width="840" alt="image" src="https://github.com/chirag99969/SPL/assets/69359027/58fc46eb-1e0d-41c0-be2e-bea20da6f5d9">

### 2.6 stats values function (unique occurences)
```
index=botsv1 sourcetype=win* EventCode=4624 
 | stats values(Account_Name) as users
 ```
 <img width="659" alt="image" src="https://github.com/chirag99969/SPL/assets/69359027/05595826-1a8e-4fa8-ac39-849392e796d3">

### 2.7 stats command values and dictinct count function 
```
index=botsv1 sourcetype=win* EventCode=4624 
 | stats values(Account_Name) as users dc(Account_Name) as "Distinct Users"
```
<img width="1751" alt="image" src="https://github.com/chirag99969/SPL/assets/69359027/626db0a1-8b37-42ae-a09f-65d1565cd86e">

 ![image](/uploads/6c3bff898bb74ef30906889a1b51219d/image.png)


