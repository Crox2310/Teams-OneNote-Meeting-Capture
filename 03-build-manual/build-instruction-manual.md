# Build Instruction Manual

## Build principles

```text
1. Build from shared contract first.
2. Build and validate Flow A before topic routing that depends on Flow A outputs.
3. Add Outlook Data Capture Profile V1 to Flow A before final PageHtml assembly.
4. Build topic branches before full Flow B recurring logic where safe.
5. Do not build Flow B recurring actions until connector validation gates pass.
6. Build Flow B recurring logic once for UJ3 and UJ4 together.
7. Never call Flow B unless the universal Flow B call gate passes.
8. Inclusion flags are Text type. Compare as strings in all topic conditions.
```

Correct:

```powerfx
Topic.IncludeLocation = "true"
```

Incorrect:

```powerfx
Topic.IncludeLocation = true
```

## Flow A

Flow name: `PA - Resolve Meeting Selection - v1 Clean Build`

Connector: `Office 365 Outlook — Get calendar view of events`

Required debug action: `FA03A_DEBUG_RawConnectorOutput`

## Flow B

Build only after connector validation gates pass.
