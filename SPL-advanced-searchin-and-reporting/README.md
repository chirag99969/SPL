# 1 Grouping and Ungrouping Data Use Case (contingency and untable command)

```
index=main sourcetype="app:logs"
| table _time emailAddress action referrer sourceIp sessionId
| eventstats earliest(referrer) as first_referrer by sessionId
| contingency usetotal=f first_referrer action
```

# 2 Grouping Multiseries data Use Case (xyseries command )

```
index=main sourcetype="app:logs"
| table _time emailAddress action referrer sourceIp sessionId
| eventstats earliest(referrer) as first_referrer by sessionId
| stats count as hits dc(emailAddress) as customers by first_referrer action
| xyseries format="$AGG$_$VAL$:action" first_referrer action hits customers
| foreach "*:*" 
    [eval <<FIELD>> = tostring('<<FIELD>>', "commas")]
```

# 3 Working with regex for Data manipulation

```
index=main sourcetype="app:logs"
| replace "" with "empty" in cart
| table _time emailAddress action referrer  sessionId cart
| stats earliest(referrer) as first_referrer, latest(action) as action, latest(cart) as cart by sessionId emailAddress
| rex field=cart max_match=0 "((?<itemId>[^\:]*)\:(?<quantity>[^\,]*),?)"
| rex field=emailAddress mode=sed "s/\..*\@/xxxx/1"
```




