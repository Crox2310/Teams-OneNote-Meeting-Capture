Full definition of the `PA - Scratch Diagnostics` flow as of 28 Aug 2026, immediately after the Teams chat extraction chain was proven working end-to-end (see `handover-2026-08-28-teams-chat-power-automate-confirmed.md` for the narrative/debugging record). This is the raw, reusable action-by-action definition — kept so this exact working pattern can be reconstructed or extended (e.g. for transcript testing) without rebuilding it from scratch or re-deriving expressions that already took several rounds of debugging to get right.

**Do not treat this as a template to blindly copy into production flows (Flow A/B).** This is a scratch/test artifact. The test meeting used throughout ("Discussion on graph access") is hardcoded into SD11's filter — swap that subject string to point at a different meeting.

**Chain summary:** Manual trigger → Get calendar view of events (V3) → SD11_FilterEvent (filter by subject) → SD12–SD15 (chained Compose actions extracting the classic-format Teams join URL out of the matched event's raw HTML body) → Get an online meeting (Teams connector, lookup by joinWebUrl → returns chatInfo.threadId) → SD16_GetChatMessages (Teams connector "Send a Microsoft Graph HTTP request", GET `/v1.0/me/chats/{threadId}/messages`).

**Key gotchas baked into this definition** (see the handover doc for full detail):
- SD11's Filter Array "Advanced mode" condition must NOT be prefixed with `@` (unlike Compose/URI fields elsewhere in this chain, which require it) — that field is already expression-typed.
- SD12–SD15 must reference `body('SD11_FilterEvent')` (not `outputs(...)`), since Filter Array's result sits under `body`, and must use `first(...)` since Filter Array always returns an array.
- "Send a Microsoft Graph HTTP request" is not a free-form HTTP caller — it validates the URL path against an explicit resource/object allow-list (resource: teams/me/users; object: channels/chats/installedApps/messages/pinnedMessages/onlineMeetings). `chats/{id}/messages` alone fails; must be `me/chats/{id}/messages`.
- "HTTP with Microsoft Entra ID" is DLP-blocked in this tenant (policy: "Copilot Studio Default Policy") — do not attempt to reintroduce it into this or related flows without a policy change from Power Platform admins.

---

## Trigger — Manually trigger a flow (Button)

```json
{
  "type": "Request",
  "kind": "Button",
  "inputs": {
    "schema": {
      "type": "object",
      "properties": {
        "location": {
          "type": "object",
          "properties": {
            "fullAddress": {
              "title": "Full address",
              "type": "string",
              "x-ms-dynamically-added": false
            },
            "address": {
              "type": "object",
              "properties": {
                "countryOrRegion": {
                  "title": "Country/Region",
                  "type": "string",
                  "x-ms-dynamically-added": false
                },
                "city": {
                  "title": "City",
                  "type": "string",
                  "x-ms-dynamically-added": false
                },
                "state": {
                  "title": "State",
                  "type": "string",
                  "x-ms-dynamically-added": false
                },
                "street": {
                  "title": "Street",
                  "type": "string",
                  "x-ms-dynamically-added": false
                },
                "postalCode": {
                  "title": "Postal code",
                  "type": "string",
                  "x-ms-dynamically-added": false
                }
              },
              "required": [
                "countryOrRegion",
                "city",
                "state",
                "street",
                "postalCode"
              ]
            },
            "coordinates": {
              "type": "object",
              "properties": {
                "latitude": {
                  "title": "Latitude",
                  "type": "number",
                  "x-ms-dynamically-added": false
                },
                "longitude": {
                  "title": "Longitude",
                  "type": "number",
                  "x-ms-dynamically-added": false
                }
              },
              "required": [
                "latitude",
                "longitude"
              ]
            }
          }
        },
        "key-button-date": {
          "title": "Date",
          "type": "string",
          "x-ms-dynamically-added": false
        }
      },
      "required": [
        "location",
        "key-button-date"
      ]
    }
  }
}
```

## Get calendar view of events (V3)

```json
{
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "calendarId": "AAMkAGY0OGU4Mzk5LWQ4NTYtNDU4MS1hY2YyLTQxOWYwZjhiMWM1ZQBGAAAAAADWkXK1vW2mQ4SwNGpyD7SzBwB8mPnOPkRmT5-MxNoNopoPAAAAAAEGAAB8mPnOPkRmT5-MxNoNopoPAACPZPifAAA=",
      "startDateTimeUtc": "@addDays(utcNow(),-3)",
      "endDateTimeUtc": "@utcNow()"
    },
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_office365",
      "connection": "shared_office365",
      "operationId": "GetEventsCalendarViewV3"
    }
  },
  "runAfter": {}
}
```

## SD11_FilterEvent

```json
{
  "type": "Query",
  "inputs": {
    "from": "@outputs('Get_calendar_view_of_events_(V3)')?['body/value']",
    "where": "@equals(item()?['subject'], 'Discussion on graph access')"
  },
  "runAfter": {
    "Get_calendar_view_of_events_(V3)": [
      "Succeeded"
    ]
  }
}
```

## SD12 — Compose BodyBeforeId

```json
{
  "type": "Compose",
  "inputs": "@substring(first(body('SD11_FilterEvent'))?['body'], 0, indexOf(first(body('SD11_FilterEvent'))?['body'], 'id=\"meet_invite_block.action.join_link_compatibility\"'))",
  "runAfter": {
    "SD11_FilterEvent": [
      "Succeeded"
    ]
  }
}
```

## SD13 — Compose UrlStart

```json
{
  "type": "Compose",
  "inputs": "@add(lastIndexOf(outputs('SD12_—_Compose_BodyBeforeId'), 'href=\"'), length('href=\"'))",
  "runAfter": {
    "SD12_—_Compose_BodyBeforeId": [
      "Succeeded"
    ]
  }
}
```

## SD14 — Compose JoinUrlRaw

```json
{
  "type": "Compose",
  "inputs": "@substring(first(body('SD11_FilterEvent'))?['body'], outputs('SD13_—_Compose_UrlStart'))",
  "runAfter": {
    "SD13_—_Compose_UrlStart": [
      "Succeeded"
    ]
  }
}
```

## SD15 — Compose JoinUrl

```json
{
  "type": "Compose",
  "inputs": "@substring(outputs('SD14_—_Compose_JoinUrlRaw'), 0, indexOf(outputs('SD14_—_Compose_JoinUrlRaw'), '\"'))",
  "runAfter": {
    "SD14_—_Compose_JoinUrlRaw": [
      "Succeeded"
    ]
  }
}
```

## Get an online meeting (Teams connector)

```json
{
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "lookupType": "joinWebUrl",
      "lookupValue": "@outputs('SD15_—_Compose_JoinUrl')"
    },
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_teams",
      "connection": "shared_teams",
      "operationId": "GetOnlineMeeting"
    }
  },
  "runAfter": {
    "SD15_—_Compose_JoinUrl": [
      "Succeeded"
    ]
  }
}
```

## SD16_GetChatMessages (Teams connector — "Send a Microsoft Graph HTTP request")

```json
{
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "Uri": "https://graph.microsoft.com/v1.0/me/chats/@{encodeURIComponent(outputs('Get_an_online_meeting')?['body']?['chatInfo']?['threadId'])}/messages",
      "Method": "GET",
      "ContentType": "application/json"
    },
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_teams",
      "connection": "shared_teams",
      "operationId": "HttpRequest"
    }
  },
  "runAfter": {
    "Get_an_online_meeting": [
      "Succeeded"
    ]
  }
}
```
