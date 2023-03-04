# BUY me a coffee ( like this ?)
## INDIAN Users (UPI ID) --- chirag99969@oksbi
## International Users (Paypal username) --- @cybersecnerd



## Splunk Installation
```
wget -O splunk-9.0.3-dd0128b1f8cd-linux-2.6-amd64.deb "https://download.splunk.com/products/splunk/releases/9.0.3/linux/splunk-9.0.3-dd0128b1f8cd-linux-2.6-amd64.deb"
mv splunk-9.0.3-dd0128b1f8cd-linux-2.6-amd64.deb /tmp
cd /tmp
sudo dpkg -i splunk-9.0.3-dd0128b1f8cd-linux-2.6-amd64.deb 
sudo /opt/splunk/bin/splunk enable boot-start --accept-license --answer-yes
sudo service splunk start
```

## sourcetype 

vendor_sales
access_combined_wcookie
secure-2
csv

# 1 Structure of SPL

## 1.1 By Sourcetype
```
index=main 
|  stats count by sourcetype
```
<img width="1792" alt="image" src="https://user-images.githubusercontent.com/69359027/221803080-39cd70b7-5494-44e8-b762-3c4324d4a458.png">


## 1.2 timechart command
```
index=main sourcetype=access_combined_wcookie status!=200 
| timechart count
```
<img width="1769" alt="image" src="https://user-images.githubusercontent.com/69359027/221803843-b8fff7f8-b6c3-47c2-9c99-1cca03c436bc.png">


## 1.3 timechart command, also grouping by value
```
index=main sourcetype=access_combined_wcookie status!=200 
| timechart count span=1h by categoryId
```
<img width="1779" alt="image" src="https://user-images.githubusercontent.com/69359027/221804307-b3dfa33a-c7a7-462c-bc68-df74746efc66.png">

# 2 Basic Splunk Searches

## 2.1 Generate a report of nunmber of events grouped by categoryId
```
index=main sourcetype=access_combined_wcookie 
| stats count by categoryId
```
<img width="1781" alt="image" src="https://user-images.githubusercontent.com/69359027/221805355-052cecc6-f81a-4472-81e8-402418203977.png">


## 2.2 generate a report of top 10 vendorIDs for code D or E
```
index=main sourcetype=vendor_sales Code=D OR Code=E
| top VendorID
```
<img width="1787" alt="image" src="https://user-images.githubusercontent.com/69359027/221807633-01a61103-06bb-4ee5-9d4a-16a7b446bec0.png">


## 2.3 Retrieve all events where AcctID starts with 6 
```
index=main sourcetype=vendor_sales AcctID=6*
```

## 2.4 Retrieve all events where response size is greater than 3500
```
index=main sourcetype="access_combined_wcookie" bytes>3500
```
<img width="1759" alt="image" src="https://user-images.githubusercontent.com/69359027/221808980-3a0da7b7-f9ed-4985-9524-93d60bef90a5.png">

## 2.5 Calculate the biggest response size 
```
index=main sourcetype="access_combined_wcookie" 
| stats max(bytes)
```
<img width="719" alt="image" src="https://user-images.githubusercontent.com/69359027/221809489-9791c021-4ad8-434b-97ac-58c82d719086.png">

## 2.6 report that shows the biggest, smallest and median size of responses and total number of requests
```
index=main sourcetype="access_combined_wcookie" 
| stats max(bytes) as BIGGEST_RESPONSE min(bytes) as SMALLEST_RESPONSE median(bytes) as MEDIAN_RESPONSE count as total_no_requests
```
<img width="1789" alt="image" src="https://user-images.githubusercontent.com/69359027/221811019-8d306812-9477-47b6-b715-5011ce540994.png">

# 3 Creating Statistics (stats)

## 3.1 stats UC1
```
index=main sourcetype="access_combined_wcookie"status!=200
|  stats count by status
```
<img width="1773" alt="image" src="https://user-images.githubusercontent.com/69359027/221812967-04227b1c-d4b6-4186-aa2b-07e5cac96560.png">


## 3.2 stats with eval UC2
```
index=main sourcetype="access_combined_wcookie" 
| stats count(eval(status =500)) as "Internal Server Errors"
```
<img width="1099" alt="image" src="https://user-images.githubusercontent.com/69359027/221817184-13a18c32-5adb-41bd-a48e-1d7eb969e624.png">


## 3.3 fieldsummary
```
index=main sourcetype="access_combined_wcookie" 
| fieldsummary maxvals=5
```
<img width="1767" alt="image" src="https://user-images.githubusercontent.com/69359027/221818383-88e88e04-f995-4334-832a-0d0f523892f1.png">

## 3.4 eventstats
```
index=main sourcetype="access_combined_wcookie" 
| timechart avg(bytes) as Response_size span=1h
| eventstats avg(Response_size)
```
<img width="1772" alt="image" src="https://user-images.githubusercontent.com/69359027/221824225-aa18c7ec-da0b-45b5-8f59-e08c52a61290.png">


## 3.5 streamstats
```
index=_internal log_level IN (WARN, ERROR)
| stats count by component log_level
| sort count 
| streamstats sum(count)
```
<img width="1761" alt="image" src="https://user-images.githubusercontent.com/69359027/221825634-459fcd07-ef44-428d-93ea-fed89b3f1b07.png">


## 3.6 streamstats (Ranking)
```
index=main sourcetype=access_combined_wcookie action=purchase
| stats count as total_purchase by itemId
| sort 5 -total_purchase 
| streamstats count as rank
```
<img width="1780" alt="image" src="https://user-images.githubusercontent.com/69359027/221832689-769a8eb3-ecac-4ce5-8a68-b67219748eed.png">


# 3 eval 

## 3.7 eval UC1
```
index=main sourcetype=access_combined_wcookie 
| eval kbytes = round(bytes/1024, 2)
```
<img width="1773" alt="image" src="https://user-images.githubusercontent.com/69359027/221836512-e4a9922b-d2ac-4237-83da-cf746c39f713.png">

## 3.8 eval UC 2 (conditional functions)
```
index=main sourcetype=secure-2
| eval result= if(like(_raw, "%Failed password%"), "failed", "success")
```
<img width="1539" alt="image" src="https://user-images.githubusercontent.com/69359027/221838655-3fc76e5f-fa76-44fe-8351-da7ea5e87de7.png">


## 3.9 eval UC 3 (case)
```
index=main sourcetype="access_combined_wcookie" 
| eval CATEGORY = case(status >= 500, "Server Error", status >=400, "Client Erorr", status = 200, "OKAY", 1=1, "N/A")
```
<img width="1769" alt="image" src="https://user-images.githubusercontent.com/69359027/221839768-cb01b9e4-7259-44c2-a501-e34d1a492db1.png">

## 3.10 eval UC 4 (string concatenation)
```
index=main sourcetype="access_combined_wcookie" 
| eval itemProduct = itemId."/".productId
```
<img width="1470" alt="image" src="https://user-images.githubusercontent.com/69359027/221842102-a8e6a39e-bf8e-49b5-aea9-19e1ec014683.png">


# 3 timechart

## 3.11 chart
```
index=main sourcetype="access_combined_wcookie" 
|  chart count by action status
```
<img width="1782" alt="image" src="https://user-images.githubusercontent.com/69359027/221845296-834653d6-ace4-4ab8-be6a-8dc12fef476d.png">

## 3.12 timchart UC1
```
index=main sourcetype="secure-2" "Failed password"
| timechart count
```
<img width="1779" alt="image" src="https://user-images.githubusercontent.com/69359027/221845938-13c4b8dc-db4c-49e5-85a9-09e9b5d0e7f4.png">

## 3.13 timechart UC 2 (timewrap) multiple time frames
```
index=_internal log_level=WARN 
| timechart count  span=1h 
| timewrap 1d
```
<img width="1790" alt="image" src="https://user-images.githubusercontent.com/69359027/221848136-4cc572fa-a7a7-4d09-ad9a-3696c8d3c880.png">

# 4 Fields and fields extraction 

## 4.1 Using time fields
```
index=main sourcetype="access_combined_wcookie" action=purchase date_wday IN (saturday, sunday) 
| stats count by date_hour
| sort date_hour 
| rename date_hour as "Hour of the day", count as "No of purchases"
```
<img width="1788" alt="image" src="https://user-images.githubusercontent.com/69359027/221855205-ada3902d-72fe-4e94-b0ea-91bf94128781.png">

## 4.2 Using the field extraction wizard
* Demo

## 4.3 Using rex command

### 4.3.1 rex command

```
index=main sourcetype="secure-2"
| rex "from (?<IP_ADDRESS>\d{1,3}.\d{1,3}.\d{1,3}.\d{1,3}")
```
<img width="1536" alt="image" src="https://user-images.githubusercontent.com/69359027/222475147-6494ecb8-7408-4d6e-a316-138d4d655444.png">

### 4.3.2 rex command

```
index=main sourcetype="secure-2"
| rex "from (?<IP_ADDRESS>[\S]+)"
```

### 4.3.3 iplocation

```
index=main sourcetype="secure-2"
| rex "from (?<IP_ADDRESS>[\S]+)"
| search IP_ADDRESS=*
| iplocation IP_ADDRESS
| stats count by Country
```

<img width="1418" alt="image" src="https://user-images.githubusercontent.com/69359027/222476021-cb77f49c-2d56-49fd-a23e-07bb02359f97.png">

### 4.3.4 geostats

```
index=main sourcetype="secure-2"
| rex "from (?<IP_ADDRESS>[\S]+)"
| search IP_ADDRESS=*
| iplocation IP_ADDRESS
| geostats count by Region
```

<img width="1768" alt="image" src="https://user-images.githubusercontent.com/69359027/222478148-fd9788ef-2ef8-455b-8062-034660cc7e7b.png">


### 4.3.5 timechart

```
index=main sourcetype="secure-2"
| rex "from (?<IP_ADDRESS>[\S]+)"
| search IP_ADDRESS=*
| iplocation IP_ADDRESS
| timechart count by Country
```
<img width="1764" alt="image" src="https://user-images.githubusercontent.com/69359027/222479068-4152b325-9021-4f86-ad71-818d298e7b30.png">


## 5 Grouping Events and using lookup

### 5.1
```
index=main sourcetype="access_combined_wcookie" 
| transaction JSESSIONID clientip
```

### 5.2
```
index=main sourcetype="access_combined_wcookie" 
| transaction JSESSIONID clientip
| where duration > 7
```

### 5.3
```
index=main sourcetype="access_combined_wcookie" 
| transaction JSESSIONID clientip
| timechart avg(duration) as "Average Session Seconds"
```

### 5.4
```
index=main sourcetype="access_combined_wcookie" 
| transaction JSESSIONID clientip
| timechart span=1h avg(duration) as "Average Session Seconds"
```
<img width="1773" alt="image" src="https://user-images.githubusercontent.com/69359027/222690767-8bca1672-4ae1-4670-96dc-49c0ed99adbf.png">

### 5.5
```
index=main sourcetype="access_combined_wcookie" 
| transaction JSESSIONID clientip
| stats max(duration) as "Longest Session"
```
<img width="989" alt="image" src="https://user-images.githubusercontent.com/69359027/222690525-c0fd14d7-2714-4634-8ada-8618716837b1.png">


### 5.6
```
index=main sourcetype="access_combined_wcookie" 
| transaction JSESSIONID clientip startswith="action=view" endswith="action=purchase" 
| stats count  values(action) values(duration) as durat by JSESSIONID _time 
| sort -durat
```
<img width="1769" alt="image" src="https://user-images.githubusercontent.com/69359027/222690285-5ac20ea7-4a7a-4963-bca0-5ecc66822ce1.png">

### 5.7 subsearch Report on itemIds for top productIds (top productIds can always be changing and unpredictable)
```
index=main sourcetype="access_combined_wcookie" 
    [ search sourcetype="access_combined_wcookie" 
    | top 5 productId 
    | fields productId ]
| stats count values(itemId) as itemId by productId
| eval itemId = mvjoin(itemId, ", ")
```
<img width="1783" alt="image" src="https://user-images.githubusercontent.com/69359027/222693774-8f76c2cb-bcb3-422f-95b3-c734552e5147.png">


### 5.8 append | comparing all yesterday sales with all time sales 
```
index=main sourcetype="access_combined_wcookie" action=purchase categoryId=ARCADE earliest="02/27/2023:00:00:00"
| stats count as "YESTERDAY Sales"
| append 
    [search sourcetype=access_combined_wcookie action=purchase categoryId=ARCADE earliest=1
    | stats count as "ALL time sales"]
```
<img width="1786" alt="image" src="https://user-images.githubusercontent.com/69359027/222697345-57430683-3bec-4ed8-98e0-795bee32c6c1.png">

### 5.9 appendcols | overlay the columns 
```
index=main sourcetype="access_combined_wcookie" action=purchase categoryId=ARCADE earliest="02/27/2023:00:00:00"
| stats count as "YESTERDAY Sales"
| appendcols
    [search sourcetype=access_combined_wcookie action=purchase categoryId=ARCADE earliest=1
    | stats count as "ALL time sales"]
```
<img width="1786" alt="image" src="https://user-images.githubusercontent.com/69359027/222697902-a07354fb-460c-46a5-9646-eadb97366eb3.png">


### 5.10 appendpipe | calculating the subtotals
```
index=main sourcetype="access_combined_wcookie" 
| top 3 itemId by productId showperc=f
| appendpipe 
    [ stats sum(count) by productId
    | eval itemId = "Total of ".productId]
| sort productId
```
<img width="1767" alt="image" src="https://user-images.githubusercontent.com/69359027/222703277-6fe3c43d-8bd2-438e-b092-c78ec412d6ad.png">

### 5.11 lookup | enriching the data
```
index=main sourcetype="access_combined_wcookie" action=purchase
| top 3 productId showperc=f
| lookup prices.csv productId
```
<img width="1776" alt="image" src="https://user-images.githubusercontent.com/69359027/222708531-3f56b3cf-ddc1-40d2-8494-970eed0aa777.png">


### 5.12 outputlookup | inputlookup 
```
index=_internal log_level=ERROR
| stats count by component
| sort -count
| head 10
| outputlookup components_error.csv
```
<img width="1771" alt="image" src="https://user-images.githubusercontent.com/69359027/222709437-dde8e67b-d32b-4111-8c26-bc8cc7dc9e73.png">

# 6 Creating Dashboards

## 6.1 Creating basic dashboards
* Demo

## 6.2 Configuring Drilldown
* Demo

## 6.3 Configuring Dropdown
* Demo 

### time range picker as dropdown menu
```
index=main sourcetype="access_combined_wcookie" action=purchase
| lookup prices.csv productId 
| timechart span=1d sum(sale_price) as Total_Revenue
```

### dropdown for products | main search

```
index=main sourcetype="access_combined_wcookie" 
| lookup prices.csv productId 
| stats sum(sale_price) as Revenue by product_name
| search product_name = "$tok_product_name$"
| sort -Revenue
```

### dropdown for products | dynamic option | search string
```
index=main sourcetype="access_combined_wcookie" action=purchase
| lookup prices.csv productId 
| stats count by product_name
```

  





