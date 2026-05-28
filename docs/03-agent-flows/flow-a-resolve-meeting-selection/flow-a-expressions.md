# Flow A v3.1 Expressions

## Value type legend

- **Expression** = enter in Power Automate expression editor.
- **Plain text** = type directly as text.
- **Dynamic content** = select from the connector/action dynamic content panel.

## FA01_Compose_StartWindowUtc

Value type: **Expression**

```powerautomate
formatDateTime(utcNow(), 'yyyy-MM-ddT00:00:00Z')
```

## FA02_Compose_EndWindowUtc

Value type: **Expression**

```powerautomate
formatDateTime(utcNow(), 'yyyy-MM-ddT23:59:59Z')
```

## FA03_O365_Get_Calendar_View_Events

| Connector field | Value | Value type |
|---|---|---|
| Calendar Id | Select default calendar in UI | Dynamic content / UI selection |
| Start time | `outputs('FA01_Compose_StartWindowUtc')` | Expression |
| End time | `outputs('FA02_Compose_EndWindowUtc')` | Expression |

## FA03A_DEBUG_RawConnectorOutput

Value type: **Expression**

```powerautomate
body('FA03_O365_Get_Calendar_View_Events')
```

## FA04_Compose_CalendarEventArray

Value type: **Expression**

```powerautomate
coalesce(body('FA03_O365_Get_Calendar_View_Events')?['value'], json('[]'))
```

## FA05_Filter_Valid_Meetings

From value type: **Expression**

```powerautomate
outputs('FA04_Compose_CalendarEventArray')
```

Advanced mode value type: **Expression**

```powerautomate
@and(
  not(empty(item()?['subject'])),
  not(equals(coalesce(item()?['isCancelled'], false), true)),
  not(equals(coalesce(item()?['isAllDay'], false), true))
)
```

If the editor rejects the leading `@`, use:

```powerautomate
and(
  not(empty(item()?['subject'])),
  not(equals(coalesce(item()?['isCancelled'], false), true)),
  not(equals(coalesce(item()?['isAllDay'], false), true))
)
```

## FA06_Compose_CandidateArray

Value type: **Expression**

```powerautomate
take(
  body('FA05_Filter_Valid_Meetings'),
  int(if(empty(triggerBody()?['InMaxCandidates']), '5', triggerBody()?['InMaxCandidates']))
)
```

## FA07_Compose_MatchCountNumber

Value type: **Expression**

```powerautomate
length(outputs('FA06_Compose_CandidateArray'))
```

## FA07A_Set_FA_VAR_MatchCount

Value type: **Expression**

```powerautomate
string(outputs('FA07_Compose_MatchCountNumber'))
```

## FA08A_Condition_MatchCount_Equals_Zero

Value type: **Expression**

```powerautomate
equals(outputs('FA07_Compose_MatchCountNumber'), 0)
```

## FA08B_Condition_MatchCount_Equals_One

Value type: **Expression**

```powerautomate
equals(outputs('FA07_Compose_MatchCountNumber'), 1)
```

## FA09_Compose_SingleMatchEvent

Value type: **Expression**

```powerautomate
first(outputs('FA06_Compose_CandidateArray'))
```

## FA09B_Set_MeetingTitle

Value type: **Expression**

```powerautomate
coalesce(outputs('FA09_Compose_SingleMatchEvent')?['subject'], '')
```

## FA09C_Set_CalendarEventId

Value type: **Expression**

```powerautomate
coalesce(outputs('FA09_Compose_SingleMatchEvent')?['id'], '')
```

## FA09D_Set_Start

Value type: **Expression**

```powerautomate
coalesce(
  outputs('FA09_Compose_SingleMatchEvent')?['start']?['dateTime'],
  ''
)
```

## FA09E_Set_End

Value type: **Expression**

```powerautomate
coalesce(
  outputs('FA09_Compose_SingleMatchEvent')?['end']?['dateTime'],
  ''
)
```

## FA09F_Set_Location

Value type: **Expression**

```powerautomate
coalesce(
  outputs('FA09_Compose_SingleMatchEvent')?['location']?['displayName'],
  ''
)
```

## FA09G_Set_IsRecurring

Value type: **Expression**

```powerautomate
if(
  or(
    equals(toLower(coalesce(outputs('FA09_Compose_SingleMatchEvent')?['type'], 'singleinstance')), 'occurrence'),
    equals(toLower(coalesce(outputs('FA09_Compose_SingleMatchEvent')?['type'], 'singleinstance')), 'exception')
  ),
  'true',
  'false'
)
```

## FA09H_Set_SeriesMasterId

Value type: **Expression**

```powerautomate
coalesce(
  outputs('FA09_Compose_SingleMatchEvent')?['seriesMasterId'],
  outputs('FA09_Compose_SingleMatchEvent')?['SeriesMasterId'],
  outputs('FA09_Compose_SingleMatchEvent')?['iCalUId'],
  ''
)
```

## FA13B_Append_To_FA_VAR_CandidateList

Value type: **Expression**

```powerautomate
concat(
  string(variables('FA_VAR_CandidateIndex')),
  '. ',
  coalesce(item()?['subject'], '(no title)'),
  ' — ',
  if(
    empty(item()?['start']?['dateTime']),
    '(no time)',
    formatDateTime(item()?['start']?['dateTime'], 'dd MMM, HH:mm')
  ),
  '–',
  if(
    empty(item()?['end']?['dateTime']),
    '(no time)',
    formatDateTime(item()?['end']?['dateTime'], 'HH:mm')
  ),
  decodeUriComponent('%0A')
)
```
