## sourcetype 

vendor_sales
access_combined_wcookie
secure-2
csv

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

## Generate a report of nunmber of events grouped by categoryId
```
index=main sourcetype=access_combined_wcookie 
| stats count by categoryId
```
<img width="1781" alt="image" src="https://user-images.githubusercontent.com/69359027/221805355-052cecc6-f81a-4472-81e8-402418203977.png">


## generate a report of top 10 vendorIDs for code D or E
```
<img width="1781" alt="image" src="https://user-images.githubusercontent.com/69359027/221807403-75896440-99fc-491f-a428-6014f68e1373.png">




