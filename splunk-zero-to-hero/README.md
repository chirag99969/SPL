## sourcetype 

vendor_sales
access_combined_wcookie
secure-2
csv

# Structure of SPL

## By Sourcetype
```
index=main 
|  stats count by sourcetype
```
<img width="1792" alt="image" src="https://user-images.githubusercontent.com/69359027/221803080-39cd70b7-5494-44e8-b762-3c4324d4a458.png">


## timechart command
```
index=main sourcetype=access_combined_wcookie status!=200 
| timechart count
```
<img width="1769" alt="image" src="https://user-images.githubusercontent.com/69359027/221803843-b8fff7f8-b6c3-47c2-9c99-1cca03c436bc.png">


## timechart command, also grouping by value
```
index=main sourcetype=access_combined_wcookie status!=200 
| timechart count span=1h by categoryId
```
<img width="1779" alt="image" src="https://user-images.githubusercontent.com/69359027/221804307-b3dfa33a-c7a7-462c-bc68-df74746efc66.png">

# Basic Splunk Searches

## Generate a report of nunmber of events grouped by categoryId
```
index=main sourcetype=access_combined_wcookie 
| stats count by categoryId
```
<img width="1781" alt="image" src="https://user-images.githubusercontent.com/69359027/221805355-052cecc6-f81a-4472-81e8-402418203977.png">


## generate a report of top 10 vendorIDs for code D or E
```
index=main sourcetype=vendor_sales Code=D OR Code=E
| top VendorID
```
<img width="1787" alt="image" src="https://user-images.githubusercontent.com/69359027/221807633-01a61103-06bb-4ee5-9d4a-16a7b446bec0.png">


## Retrieve all events where AcctID starts with 6 
```
index=main sourcetype=vendor_sales AcctID=6*
```

## Retrieve all events where response size is greater than 3500
```
index=main sourcetype="access_combined_wcookie" bytes>3500
```
<img width="1759" alt="image" src="https://user-images.githubusercontent.com/69359027/221808980-3a0da7b7-f9ed-4985-9524-93d60bef90a5.png">

## Calculate the biggest response size 
```
index=main sourcetype="access_combined_wcookie" 
| stats max(bytes)
```
<img width="719" alt="image" src="https://user-images.githubusercontent.com/69359027/221809489-9791c021-4ad8-434b-97ac-58c82d719086.png">

## report that shows the biggest, smallest and median size of responses and total number of requests
```
index=main sourcetype="access_combined_wcookie" 
| stats max(bytes) as BIGGEST_RESPONSE min(bytes) as SMALLEST_RESPONSE median(bytes) as MEDIAN_RESPONSE count as total_no_requests
```
<img width="1789" alt="image" src="https://user-images.githubusercontent.com/69359027/221811019-8d306812-9477-47b6-b715-5011ce540994.png">

# Creating Statistics (stats)

## stats UC1
```
index=main sourcetype="access_combined_wcookie"status!=200
|  stats count by status
```
<img width="1773" alt="image" src="https://user-images.githubusercontent.com/69359027/221812967-04227b1c-d4b6-4186-aa2b-07e5cac96560.png">


## stats with eval UC2
```
index=main sourcetype="access_combined_wcookie" 
| stats count(eval(status =500)) as "Internal Server Errors"
```
<img width="1099" alt="image" src="https://user-images.githubusercontent.com/69359027/221817184-13a18c32-5adb-41bd-a48e-1d7eb969e624.png">


## fieldsummary
```
index=main sourcetype="access_combined_wcookie" 
| fieldsummary maxvals=5
```
<img width="1767" alt="image" src="https://user-images.githubusercontent.com/69359027/221818383-88e88e04-f995-4334-832a-0d0f523892f1.png">

## eventstats
```
index=main sourcetype="access_combined_wcookie" 
| timechart avg(bytes) as Response_size span=1h
| eventstats avg(Response_size)
```
<img width="1772" alt="image" src="https://user-images.githubusercontent.com/69359027/221824225-aa18c7ec-da0b-45b5-8f59-e08c52a61290.png">


## streamstats
```
index=_internal log_level IN (WARN, ERROR)
| stats count by component log_level
| sort count 
| streamstats sum(count)
```
<img width="1761" alt="image" src="https://user-images.githubusercontent.com/69359027/221825634-459fcd07-ef44-428d-93ea-fed89b3f1b07.png">


## streamstats (Ranking)
```
index=main sourcetype=access_combined_wcookie action=purchase
| stats count as total_purchase by itemId
| sort 5 -total_purchase 
| streamstats count as rank
```
<img width="1780" alt="image" src="https://user-images.githubusercontent.com/69359027/221832689-769a8eb3-ecac-4ce5-8a68-b67219748eed.png">


# eval 

## eval UC1
```
index=main sourcetype=access_combined_wcookie 
| eval kbytes = round(bytes/1024, 2)
```
<img width="1773" alt="image" src="https://user-images.githubusercontent.com/69359027/221836512-e4a9922b-d2ac-4237-83da-cf746c39f713.png">

## eval UC 2 (conditional functions)
```
index=main sourcetype=secure-2
| eval result= if(like(_raw, "%Failed password%"), "failed", "success")
```
<img width="1539" alt="image" src="https://user-images.githubusercontent.com/69359027/221838655-3fc76e5f-fa76-44fe-8351-da7ea5e87de7.png">








