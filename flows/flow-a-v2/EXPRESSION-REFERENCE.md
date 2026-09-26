# Flow A v2 — Expression Reference

Quick reference for all trigger input expressions used throughout the build.
Refer to this whenever an instruction says "DateContext" or "InSelectedNumber".

| Input | Key | Expression |
|---|---|---|
| InSelectedNumber | `text` | `triggerBody()?['text']` |
| DateContext | `text_1` | `triggerBody()?['text_1']` |

## Corrected expressions by action

### CA01 Compose DateContext
```
coalesce(triggerBody()?['text_1'], utcNow())
```

### CA02 Compose StartOfDay
```
formatDateTime(if(empty(trim(coalesce(outputs('CA01_Compose_DateContext'), ''))), utcNow(), outputs('CA01_Compose_DateContext')), 'yyyy-MM-ddT00:00:00Z')
```

### CA03 Compose EndOfDay
```
formatDateTime(if(empty(trim(coalesce(outputs('CA01_Compose_DateContext'), ''))), utcNow(), outputs('CA01_Compose_DateContext')), 'yyyy-MM-ddT23:59:59Z')
```

### MM02 Get Mapping Rows — Filter Query
```
OccurrenceDate eq '@{formatDateTime(outputs('CA01_Compose_DateContext'), 'yyyy-MM-dd')}'
```

### CR02 Compose Display Date
```
formatDateTime(outputs('CA01_Compose_DateContext'), 'ddd d MMM yyyy')
```

All other expressions in the build instructions are unaffected by the key change — they reference `outputs('ActionName')` rather than trigger inputs directly.
