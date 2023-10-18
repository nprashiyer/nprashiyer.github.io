count - count the number of rows
```
StormEvents | count
```

project - select columns
```
StormEvents
| project State, EventType, DamageProperty
```

where - filter && sort
```
StormEvents
| where State == 'TEXAS' and EventType == 'Flood'
| sort by DamageProperty
| project StartTime, EndTime, State, EventType, DamageProperty
```